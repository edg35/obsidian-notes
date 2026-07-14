## Context

Today, when a user replies in a Teams 1:1 chat to a ticket alert card, the bot just echoes the text back (`src/index.ts:29-48`) — there is no ticket-awareness at all. The `llmService.ts` LLM wrapper exists but has zero callers, supports only single-turn JSON-mode completions (no tool calling), and Autotask access is **read-only** (`autotaskApiCalls.ts` has no write endpoints). There's also no way today to know *which ticket* a Teams reply is about — notification cards are broadcast to every registered conversation with no ticket ID attached.

This plan adds: a tool-calling AI agent that reads a user's Teams reply, decides what to do about the associated ticket (change status — including closing it, add an internal note, reply to the user, or do nothing), and can either execute those actions for real or run in a dry-run mode that evaluates the full decision but performs zero side effects, recording what would have happened to an audit table instead.

Two product decisions from the user, both binding on this design:
1. **Dry run is log-only.** No Teams message is sent during a dry run (not even a tagged one) — the decision is fully evaluated and durably recorded in a new audit table, with zero visible effect on the conversation.
2. **No separate "close" tool.** A single `update_ticket_status` tool exposes *all* status names from `autotaskInfo.ts`'s `statuses` map (including closing-type statuses like `Complete`/`Resolved`) and the model picks whichever is appropriate — the enum is not narrowed.

## Schema changes

New migration (`ai-helpdesk-app`: run `npm run migrate:create -- add-ticket-agent-support`, which will generate the real timestamp-prefixed filename per the existing SQL-migration convention):

```sql
-- Up Migration
ALTER TABLE conversation_messages
    ADD COLUMN IF NOT EXISTS ticket_id INT REFERENCES tickets(ticket_id) ON DELETE SET NULL;

CREATE INDEX IF NOT EXISTS idx_conversation_messages_ticket_lookup
    ON conversation_messages (conversation_id, content_type, ticket_id, created_at DESC);

CREATE TABLE IF NOT EXISTS ticket_agent_actions (
    action_id SERIAL PRIMARY KEY,
    conversation_id INT NOT NULL REFERENCES conversations(conversation_id) ON DELETE CASCADE,
    ticket_id INT REFERENCES tickets(ticket_id) ON DELETE SET NULL,
    reply_text TEXT NOT NULL,
    tool_calls JSONB NOT NULL,
    execution_results JSONB,
    dry_run BOOLEAN NOT NULL DEFAULT FALSE,
    llm_deployment VARCHAR(100),
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_ticket_agent_actions_ticket_id ON ticket_agent_actions (ticket_id);
CREATE INDEX IF NOT EXISTS idx_ticket_agent_actions_conversation_id ON ticket_agent_actions (conversation_id);
CREATE INDEX IF NOT EXISTS idx_ticket_agent_actions_created_at ON ticket_agent_actions (created_at DESC);

-- Down Migration
DROP TABLE IF EXISTS ticket_agent_actions;
DROP INDEX IF EXISTS idx_conversation_messages_ticket_lookup;
ALTER TABLE conversation_messages DROP COLUMN IF EXISTS ticket_id;
```

`ticket_id` on `conversation_messages` is nullable (existing rows/plain text messages unaffected) and uses `ON DELETE SET NULL` rather than `CASCADE`, since `newTicketNotification.ts` already deletes `tickets` rows when Autotask no longer has the ticket — that shouldn't wipe chat history. `execution_results`/`error_message` on `ticket_agent_actions` make the audit row genuinely useful for reviewing dry-run decisions (not just a bare `tool_calls` blob).

## Ticket-association strategy

**Researched and confirmed (not an assumption):** Microsoft Teams does not support quote/reply linkage to a specific prior message in 1:1 personal chats — even though the Bot Framework `Activity.replyToId` field exists, Microsoft's own guidance confirms it is delivered to Teams as a new message, not a threaded reply, in personal conversations (that threading model only exists in channels). So a plain typed reply carries **no metadata** telling us which card it's "about." However, Adaptive Card `Action.Execute` buttons *do* reliably round-trip a `data` payload: this repo's `@microsoft/teams.apps`/`@microsoft/teams.api` SDK versions have first-class support for it — an `Action.Execute` click is delivered as an `adaptiveCard/action` invoke activity with `activity.value.action.data` (a `Record<string, any>`, confirmed in `node_modules/@microsoft/teams.api/dist/models/adaptive-card/adaptive-card-invoke-action.d.ts`), and the app router already exposes a typed `card.action.<name>` event for it (`node_modules/@microsoft/teams.apps/dist/router/router.js:94-95` dispatches `card.action.${activity.value.action.data.action}`).

This gives a two-tier association strategy: an explicit, reliable signal when the user engages with a button, and the original heuristic as a fallback for users who just type directly.

**Tier 1 — explicit selection via card button (new, primary mechanism):**
1. `buildTicketAlertCard` (duplicated today in both [newTicketNotification.ts:8-83](ai-helpdesk-app/src/jobs/newTicketNotification.ts:8) and `automatedTicketNotification.ts`) gains a second card action alongside the existing `Action.OpenUrl`:
   ```json
   { "type": "Action.Execute", "title": "I have an update on this ticket", "verb": "selectTicket", "data": { "action": "select_ticket", "ticketId": ticket.ticket_id } }
   ```
2. New route registered in `src/index.ts`: `teamsApp.on('card.action.select_ticket', async ({ activity, send }) => { ... })`. Handler reads `activity.value.action.data.ticketId`, records it as the conversation's current ticket selection, and returns an invoke response acknowledging the click (Adaptive Card `Action.Execute` invokes require a response body per the Universal Actions protocol — e.g. replacing the card with a short confirmation message like "Got it — go ahead and tell me the update for ticket #1234."). Recording the selection reuses the same mechanism as tier 2 below: insert a `conversation_messages` row with `ticket_id` set and a new `content_type = 'ticket_selection'` value (no schema change needed beyond the `ticket_id` column already planned — `content_type` has no CHECK constraint, it's a free-form `VARCHAR(50)`).
3. This is deliberately a **context-setter, not an action-trigger** — clicking the button doesn't itself change the ticket; it just tells the agent which ticket the *next* free-text message is about. The actual decision-making still happens off the subsequent `message` event, per the flow already in this plan.

**Tier 2 — fallback heuristic (unchanged from the original design, still required):** if the user never clicks the button and just types directly under an alert card, there's still no Teams-provided linkage, so fall back to "most recent ticket-tagged row in this conversation." Thread an optional `ticketId` through `addCardToConversation`/`sendCardToConversation` in [conversations.ts](ai-helpdesk-app/src/conversations.ts:49) so cards keep recording their ticket; the [newTicketNotification.ts:125](ai-helpdesk-app/src/jobs/newTicketNotification.ts:125) call site passes `ticket.ticket_id` (update the disabled `automatedTicketNotification.ts` call site the same way for consistency, without touching its commented-out scheduler registration).

**Combined resolution query** (an explicit selection naturally outranks an older card simply by being more recent):

```sql
SELECT cm.ticket_id
FROM conversation_messages cm
JOIN conversations c ON cm.conversation_id = c.conversation_id
WHERE c.teams_conversation_id = $1
  AND cm.content_type IN ('card', 'ticket_selection')
  AND cm.ticket_id IS NOT NULL
  AND cm.created_at >= NOW() - ($2 || ' hours')::interval
ORDER BY cm.created_at DESC
LIMIT 1
```

- `$2` is a new `TICKET_AGENT_CONTEXT_MAX_AGE_HOURS` env var (default `72`) — a staleness guard so a reply in an otherwise-dormant chat doesn't resurrect a months-old ticket. Consider a shorter effective window for `ticket_selection` rows specifically if testing shows the button is used well before the follow-up text (easy to tune later; not a blocker for v1).
- **No match found:** skip the agent entirely and fall through to the existing echo behavior. Zero LLM cost for what's likely unrelated small talk, and it's the smallest-blast-radius choice for a first version.
- **No status filtering** at the SQL level — always use the most recent ticket-tagged row regardless of that ticket's current status. The ticket's current status is included in the LLM prompt, so the model itself can decide `no_action` on an already-closed ticket, or legitimately reopen one if the user says it broke again. A hard SQL filter can't make that judgment call and would need to duplicate a "terminal status" list that isn't the source of truth.
- **Known v1 limitation, now much smaller in practice:** without the button, "most recent card" can still pick the wrong ticket if a chat got alerts for multiple tickets and the user replies about an older one — but a user who clicks "I have an update on this ticket" first removes that ambiguity entirely. No attempt is made to parse a ticket number out of free text; that remains a reasonable future enhancement, not required now.

## New Autotask write capabilities

Add to [autotaskApiCalls.ts](ai-helpdesk-app/src/autotaskApiCalls.ts), following the file's existing conventions exactly (env var checks → build URL → fetch with `ApiIntegrationCode`/`UserName`/`Secret` headers → `verboseAutotaskLogs`-gated logging):

- `updateTicketStatus(ticketId: number, statusId: number): Promise<void>` — `PATCH https://webservices${region}.autotask.net/atservicesrest/v1.0/Tickets` with body `{ id: ticketId, status: statusId }`.
- `createTicketNote(ticketId: number, title: string, description: string): Promise<void>` — `POST .../TicketNotes` with body including `ticketID`, `title`, `description`, plus `noteType`/`publish` fields.

**Flagged unknowns to verify against Autotask's actual REST API reference before merging** (not guessed further here): the exact verb/path for ticket updates (collection-level `PATCH` with id-in-body is the REST convention Autotask generally follows, but must be confirmed), and the correct `noteType`/`publish` enum values for an *internal-only* note (existing `getTicketNotes()` already filters out `noteType: 13`, so these values are instance-meaningful and shouldn't be guessed).

**Status name → ID resolution:** the tool exposes status *names* (per the product decision, sourced from `autotaskInfo.ts`'s full `statuses` map). When executing, resolve name → ID via `SELECT status_id FROM ticket_statuses WHERE name = $1` (the DB table, refreshed daily from live Autotask data) first, falling back to a reverse lookup in the static `autotaskInfo.ts` map only if the DB has no row yet.

## Tool-calling LLM support

Extend [llmService.ts](ai-helpdesk-app/src/services/llmService.ts) with a new function alongside `callLlmJson`, reusing the existing `getClient()`/`GPT4O`/`GPT4O_MINI`:

```ts
export type LlmToolCallResult = { content: string | null; toolCalls: ChatCompletionMessageToolCall[] };

export async function callLlmWithTools(
  deployment: string, systemPrompt: string, userContent: string,
  tools: ChatCompletionTool[], toolChoice: 'auto' | 'required' = 'required'
): Promise<LlmToolCallResult> {
  const client = getClient();
  const response = await client.chat.completions.create({
    model: deployment,
    messages: [{ role: 'system', content: systemPrompt }, { role: 'user', content: userContent }],
    temperature: 0.2,
    tools,
    tool_choice: toolChoice,
  });
  const message = response.choices[0]?.message;
  if (!message) throw new Error('LLM returned empty response');
  return { content: message.content ?? null, toolCalls: message.tool_calls ?? [] };
}
```

Default `tool_choice: 'required'` so every reply produces at least one decision (even if it's just `no_action`), rather than a silent empty response.

**Single-turn, not a multi-turn agentic loop.** Ticket fields, recent notes, and the reply text are all placed directly into the prompt up front (fetched via existing `getTicketById`-style DB joins and `getTicketNotes`), so the model never needs to call a "read" tool and get results fed back. One `callLlmWithTools` call maps to exactly one Teams reply and one audit row — simple, bounded, and doesn't preclude adding a real loop later if a future feature needs the model to fetch more data mid-decision.

## Tool definitions — `src/services/ticketAgentTools.ts`

Four tools, each with a handler of shape `(args, ctx: { ticketId, teamsConversationId, dryRun }) => Promise<{ tool, dryRun, success, detail }>`:

- **`update_ticket_status`** — `{ statusName: enum(all names from autotaskInfo.ts statuses), reason: string }`. Live: resolves ID and calls `updateTicketStatus` + `createTicketNote(ticketId, 'Status changed by AI agent', reason)`. Dry run: returns `{ wouldSetStatus: statusName, reason }`, no Autotask/DB call.
- **`add_ticket_note`** — `{ noteText: string }`. Live: `createTicketNote`. Dry run: `{ wouldAddNote: noteText }`.
- **`reply_to_user`** — `{ message: string }`. Live: `sendMessageToConversation(teamsConversationId, message)`. Dry run: `{ wouldReply: message }` — `sendMessageToConversation` is never called, matching the "no Teams message during dry run" decision.
- **`no_action`** — `{ reason: string }`. Always a no-op in both modes; exists so the model has an explicit way to decline acting on irrelevant/ambiguous replies instead of guessing.

The model may return multiple tool calls in one response (e.g. `update_ticket_status` + `reply_to_user` together for "closing this out and letting the user know"). The orchestrator executes them **in the order returned**, sequentially (not in parallel), each in its own try/catch so one failing call doesn't block the others or the audit write.

## Orchestrator — `src/services/ticketReplyAgent.ts`

```ts
export type TicketReplyAgentResult =
  | { handled: false }
  | { handled: true; ticketId: number; dryRun: boolean; toolCalls: ChatCompletionMessageToolCall[] };

export async function handleTicketReply(params: {
  teamsConversationId: string; replyText: string; senderName: string;
}): Promise<TicketReplyAgentResult>
```

Flow:
1. Resolve ticket in context (query above). No match → return `{ handled: false }` (no LLM call, no audit row).
2. Load ticket fields (joined with status/priority/queue names) + recent notes via `getTicketNotes`.
3. Build system/user prompt: system describes the four tools and instructs the model to prefer `no_action` over guessing; user content is the ticket fields, recent notes, and raw reply text.
4. `dryRun = process.env.TICKET_AGENT_DRY_RUN === 'true'`; `deployment = process.env.TICKET_AGENT_LLM_DEPLOYMENT ?? GPT4O` (default to the full model, not mini, given closing tickets is consequential).
5. Call `callLlmWithTools`, wrapped in try/catch.
6. Execute each returned tool call sequentially via its handler, collecting `{tool, dryRun, success, detail}` results (a failing handler yields `success:false` rather than throwing out of the loop).
7. Write one row to `ticket_agent_actions` (conversation_id resolved via the conversations table, ticket_id, reply_text, raw tool_calls, execution_results, dry_run, llm_deployment). On an upstream failure (step 5 throws), still write a row with `tool_calls: []` and `error_message` populated, and return `{handled: true, ticketId, dryRun, toolCalls: []}` rather than rethrowing — so `index.ts` doesn't fall back to the echo response on top of a partially-run agent.
8. Return `{ handled: true, ticketId, dryRun, toolCalls }`.

## Wiring into `src/index.ts`

In the `teamsApp.on('message', ...)` handler ([index.ts:29-48](ai-helpdesk-app/src/index.ts:29)): for `activity.conversation.conversationType === 'personal'` messages, call `handleTicketReply` after registering the conversation/logging the inbound message. If `handled: true`, return (the agent already sent, or in dry-run deliberately withheld, any reply, and wrote the audit row). Otherwise fall through to the existing `You said: ...` echo unchanged — this preserves current behavior for group/channel messages and for personal messages with no ticket in context.

New route: `teamsApp.on('card.action.select_ticket', async ({ activity, send }) => { ... })`, registered alongside the existing `message`/`install.add`/`install.remove` handlers, implementing Tier 1 of the ticket-association strategy above — extracts `ticketId` from `activity.value.action.data`, records the selection (new `recordTicketSelection(teamsConversationId, ticketId)` function in `conversations.ts`, inserting a `content_type = 'ticket_selection'` row), and returns a short confirmation.

Add a `TICKET_AGENT_ENABLED` env var (default falsy) gating the whole agent call, independent of `TICKET_AGENT_DRY_RUN`, so rollout can go disabled → enabled+dry-run → enabled+live without code changes between stages. This is a low-cost addition given this ships the first-ever Autotask write path in the app.

## Test plan

All new/updated test files live under `ai-helpdesk-app/tests/`, following the conventions already established in [llmService.test.ts](ai-helpdesk-app/tests/services/llmService.test.ts) (mock external modules via `vi.mock` before importing the module under test, reset relevant `process.env` in `beforeEach`, flat `describe` per module, plain-English `it()` names) and [businessHours.test.ts](ai-helpdesk-app/tests/businessHours.test.ts)'s env snapshot/restore pattern where env vars are heavily involved.

1. **Extend `tests/services/llmService.test.ts`** — `callLlmWithTools` passes `tools`/`tool_choice` through unchanged; parses multiple `tool_calls` in order; returns `toolCalls: []` (no throw) when the mock has none; throws when `message` itself is missing.

2. **New `tests/services/ticketReplyAgent.test.ts`** — mocks `db`, `llmService.callLlmWithTools`, `autotaskApiCalls`, and `conversations.sendMessageToConversation`:
   - No ticket in context → `{handled:false}`, `callLlmWithTools` never called.
   - Ticket in context (single card match) → prompt passed to `callLlmWithTools` contains ticket fields + reply text.
   - A `ticket_selection` row more recent than a `card` row for the same conversation is the one picked (verifies the resolver's `content_type IN ('card','ticket_selection')` + `ORDER BY created_at DESC` — an explicit button click outranks a stale card).
   - Multiple tool calls in one response → both handlers run in order, one audit row written containing both.
   - Dry-run mode → Autotask/Teams calls never happen; audit row still written with `dry_run: true` and populated `execution_results`.
   - Live mode → real calls happen; audit row has `dry_run: false`.
   - LLM throws → audit row written with `error_message` set; `handleTicketReply` does not throw.

3. **New `tests/services/ticketAgentTools.test.ts`** — one block per handler (`update_ticket_status`, `add_ticket_note`, `reply_to_user`, `no_action`), each asserting dry-run skips the real side effect while live mode performs it; status handler additionally tests DB-lookup-found vs. DB-lookup-missing (falls back to the static `autotaskInfo.ts` map) status-ID resolution.

4. **New `tests/autotaskApiCalls.test.ts`** (first test file for this module) — mocks `fetch`; verifies `updateTicketStatus`/`createTicketNote` send the expected method/URL/body/headers and throw `'Missing Autotask environment variables'` when secrets are absent, matching existing functions' behavior.

5. **New/extended `tests/conversations.test.ts`** — verifies `addCardToConversation`/`sendCardToConversation` store the passed `ticketId` (and store `NULL` when omitted, for backward compatibility with existing call sites); verifies the new `recordTicketSelection` inserts a `content_type = 'ticket_selection'` row with the given `ticket_id`.

6. **New `tests/cardActionSelectTicket.test.ts`** (or colocated with an `index.ts` route-registration test if one exists) — mocks `conversations.recordTicketSelection` and the Teams `send` callback; simulates an `adaptiveCard/action` invoke with `value.action.data = { action: 'select_ticket', ticketId: 123 }`, asserts `recordTicketSelection` is called with the right conversation/ticket ID, and asserts a confirmation response is returned/sent.

7. **New, opt-in `tests/services/ticketReplyAgent.llmEval.test.ts`** — hits the **real** Azure OpenAI deployment (no `vi.mock('openai', ...)`), every case gated by `it.skipIf(!process.env.RUN_LLM_EVALS)`, so it never runs in normal `npm test`/CI. Scenarios: "yes I finished it, solution was X" → expects a closing-type `update_ticket_status`; "still waiting to hear back from the vendor" → expects a waiting-type status or note; "thanks!" on an already-closed ticket → expects `no_action`; a reply with both a fix and a follow-up question → expects both `update_ticket_status` and `reply_to_user` together. Assert on the *category* of decision, not exact wording, to avoid over-fitting to prompt phrasing.

## Verification

- `cd ai-helpdesk-app && npm test` — all new/extended Vitest files pass, and existing tests (`businessHours.test.ts`, `llmService.test.ts`) remain green.
- Apply the new migration locally (`npm run migrate:up`) against a dev Postgres instance and confirm `conversation_messages.ticket_id` and `ticket_agent_actions` exist with the expected columns/indexes.
- Manual end-to-end check with `TICKET_AGENT_ENABLED=true, TICKET_AGENT_DRY_RUN=true`: trigger (or simulate) a ticket alert card to a test Teams personal chat, reply with a realistic message, and confirm (a) no Teams reply is sent, (b) a row appears in `ticket_agent_actions` with the correct `ticket_id` and a sensible `tool_calls`/`execution_results` payload, (c) no Autotask write actually occurred (ticket unchanged in Autotask).
- Repeat with `TICKET_AGENT_DRY_RUN=false` against a disposable/test Autotask ticket and confirm the real status change / note / Teams reply happens as decided.
- Run the opt-in `RUN_LLM_EVALS=true npm test -- ticketReplyAgent.llmEval` against the real Azure OpenAI deployment to sanity-check tool selection on the representative scenarios before considering this feature complete.