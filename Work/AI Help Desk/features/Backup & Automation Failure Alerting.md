## Context

Backup failure emails (Datto) and per-client dotnet automation runs currently get triaged outside this system: a Rewst flow watches the shared inbox, parses failure emails, and posts a Teams card straight to a "backups" channel. There's no record of these events in our own database, no dedup (a re-sent failure email or a job covering many devices can spam multiple messages), and no link into the existing Autotask ticket workflow that techs already work out of.

The goal: move ingestion into our backend so these events become first-class data we can reason about, dedup, and — following the same AI tool-calling pattern already built for ticket replies (`ticketReplyAgent.ts` / `ticketAgentTools.ts`) — let an LLM decide what to do with each incoming failure (open a new ticket, update an existing open one, or take no action), then notify Teams.

Decisions made during brainstorming:
- **Ticket-first, AI-driven**: the LLM decides create-vs-update-vs-no-action, same pattern as the existing ticket-reply orchestrator.
- **Phased Autotask integration**: Phase 1 stages "tickets" in our own DB only (no real Autotask calls) so we can observe and tune the create/dedup logic risk-free. Phase 2 (separate follow-up, not in this plan's scope) swaps the staging table for real `Autotask.createTicket` calls once we're happy with the behavior.
- **Async processing**: ingestion endpoints just validate + persist + enqueue via the existing `pg-boss` job pattern (`jobScheduler.ts`), then a worker does the LLM/Teams work. Keeps Rewst's/the script's HTTP call fast and gives retries for free.
- **Dedup key = the persistent underlying job**, not a single run, so repeat/unresolved failures update the same open record instead of spamming new ones:
  - Backups: keyed on the **overall backup appliance** (serial). One open record per appliance, listing whichever downstream devices are currently failing; updated as devices join/clear across days.
  - Automation: keyed on **(client, script name)**.
- **No Teams routing/filtering** — broadcast every alert to all currently-registered `personal_chats`/`team_channels`, same as today's ticket notifications. A separate planned feature will handle who sees what.
- **No new auth exists on this Express app today** (confirmed: no API keys, no signature checks anywhere). The two new ingestion endpoints are the first internet-facing, non-Teams-authenticated routes, so they need a shared-secret check.

## Architecture

```
Rewst (backup email parser) ──POST /api/alerts/backup──┐
dotnet automation script ─────POST /api/alerts/automation┤──▶ validate + shared-secret auth
                                                          │──▶ insert alert_events row
                                                          │──▶ pg-boss.send('process-alert', {alertEventId})
                                                          └──▶ 202 Accepted (fast ack)

pg-boss worker 'process-alert' (jobScheduler.ts)
    │
    ▼
alertTriageAgent.ts
    │  1. load alert_events row
    │  2. look up open alert_tickets row by dedup_key (deterministic, in code — not left to the LLM)
    │  3. build system prompt with: new alert payload + existing open record (if any) + its history
    │  4. run callLlmWithToolsConversation loop (reuses llmService.ts, same shape as ticketReplyAgent)
    │  5. execute whichever tools the LLM calls
    │  6. mark alert_events.processed = true (or store error)
    ▼
alertAgentTools.ts tools:
  - create_ticket      → INSERT alert_tickets (status='open'), Phase 1 only (no Autotask call)
  - append_alert        → UPDATE existing open alert_tickets row (merge device/error info, bump occurrence_count/last_alert_at)
  - send_teams_alert    → buildFailureAlertCard(...) + fan out via sendCardToConversation (same loop as newTicketNotification.ts)
  - no_action           → reuse existing pattern, for malformed/non-actionable payloads
```

## Data model (new migration, `database/migrations/<ts>_add-alert-ingestion.sql`)

Follow the existing `node-pg-migrate` SQL convention (see `1781872475921_init-schema.sql`). Two tables:

**`alert_events`** — append-only raw log of every inbound alert (audit trail + retry source):
- `alert_event_id` PK
- `source` (`'backup' | 'automation'`)
- `dedup_key` text (computed at ingestion time — `backup:<applianceSerial>` or `automation:<client>:<scriptName>`)
- `raw_payload` jsonb
- `received_at` timestamptz default now()
- `processed` boolean default false
- `processing_error` text nullable

**`alert_tickets`** — the Phase-1 staged "ticket" list (becomes Phase 2's link to real Autotask tickets later):
- `alert_ticket_id` PK
- `dedup_key` text
- `source` (`'backup' | 'automation'`)
- `status` (`'open' | 'resolved'`)
- `autotask_ticket_id` integer nullable — unused in Phase 1, reserved for Phase 2
- `summary` text — LLM-written human-readable title
- `details` jsonb — structured payload, e.g. for backups `{ applianceSerial, applianceModel, devices: [{ name, reason, lastFailedAt, gentime }, ...] }`
- `occurrence_count` integer default 1
- `created_at`, `updated_at`, `last_alert_at` timestamptz
- Partial unique index: `UNIQUE (dedup_key) WHERE status = 'open'` — enforces at most one open record per job, while keeping resolved history around.

Resolution (closing an `alert_tickets` row) is out of scope for this plan's automatic wiring (that's Phase 2, via the existing `ticketUpdate` job watching Autotask status). For Phase 1 testing, add a minimal manual close: `POST /api/alerts/tickets/:id/resolve` (simple status flip, same auth as the ingestion endpoints) so you can exercise the dedup logic end-to-end without waiting on Phase 2.

## Ingestion endpoints (`ai-helpdesk-app/src/index.ts`, alongside existing routes)

Both endpoints follow the existing route style (`expressApp.post(path, express.json(), handler)`, try/catch → 500) and share a small `requireAlertAuth` middleware that checks a shared-secret header (new env var, e.g. `ALERT_INGEST_SHARED_SECRET`) against `X-Alert-Secret` — 401 on mismatch/missing.

- **`POST /api/alerts/backup`** — body: `{ appliance: { serial, model }, device: { name }, failedAt, gentime, reason }`. Compute `dedup_key = backup:${appliance.serial}`, insert `alert_events`, enqueue `process-alert`.
- **`POST /api/alerts/automation`** — body: `{ client, scriptName, status, errorDetails, timestamp }`. Compute `dedup_key = automation:${client}:${scriptName}`, insert `alert_events`, enqueue `process-alert`. (Only enqueue for failing `status`; a healthy run summary can be accepted and stored but skipped from triage — decide the exact "is this a failure" check when wiring the real payload shape.)

Both respond `202 { alertEventId }` once persisted+enqueued.

## Orchestrator: `alertAgentTools.ts` + `alertTriageAgent.ts`

Mirrors `ticketAgentTools.ts` / `ticketReplyAgent.ts` exactly in shape (tool schema + executor map, `AgentToolContext`-style context, `callLlmWithToolsConversation` loop capped at `maxIterations`).

- `create_ticket` tool: args `{ summary, details }` → executor inserts new `alert_tickets` row (Phase 1: DB only, no Autotask call — matches the `dryRun`-style branching already used in `executeUpdateTicketStatus` etc., just permanently "off" until Phase 2 flips it via an env flag).
- `append_alert` tool: args `{ additionalDetails, note }` → executor updates the existing open row (merge `details`, `occurrence_count += 1`, `last_alert_at = now()`).
- `send_teams_alert` tool: args `{ headline, urgent }` → executor builds the adaptive card and fans out to `getRegisteredPersonalChats()` + `getRegisteredTeamChannels()` via `sendCardToConversation`, same loop structure as `newTicketNotification.ts`'s notify function.
- `no_action` tool: reused as-is in spirit from `ticketAgentTools.ts`.

`alertTriageAgent.ts` does the **deterministic** dedup lookup (query `alert_tickets` for an open row matching `dedup_key`) before calling the LLM, and passes the result into the system prompt as context — the LLM is told whether this is a new job failure or an update to a known one, and decides `create_ticket` vs `append_alert` vs `no_action`, plus whether/how to call `send_teams_alert`. This mirrors how `ticketReplyAgent.ts` builds a ticket-specific system prompt before starting the tool loop.

## Card building

New `buildFailureAlertCard(alertTicket)` in a new `src/jobs/alertNotification.ts` (or colocated with the tools file), following the exact `AdaptiveCard` structure in `newTicketNotification.ts`'s `buildTicketAlertCard` (Container/TextBlock/FactSet). FactSet surfaces source, appliance/client, job/script name, reason/error, occurrence count. No `Action.OpenUrl` until Phase 2 provides a real Autotask ticket link.

## Job registration (`jobScheduler.ts`)

Add a `process-alert` queue/worker (event-driven via `boss.send(...)` from the ingestion route, not cron-scheduled like the existing jobs) calling `alertTriageAgent.ts`. No changes to the existing cron-based jobs.

## Testing

Mirror the existing test patterns:
- `tests/services/alertAgentTools.test.ts` — unit tests per tool executor (create/append/send/no_action), dryRun-style and dedup edge cases.
- `tests/services/alertTriageAgent.deterministic.test.ts` — fixture-driven, following `ticketReplyAgent.deterministic.test.ts` + `tests/fixtures/ticketReplyCards.ts` conventions; cover: first failure (create), repeat failure same appliance (append), different appliance (separate create), malformed payload (no_action).
- Endpoint tests for `/api/alerts/backup` and `/api/alerts/automation`: auth rejection, payload validation (400), happy path enqueues job and returns 202.

## Explicitly out of scope for this plan (Phase 2, later)

- Real Autotask `createTicket` API call (extending `autotaskApiCalls.ts`) and wiring `alert_tickets.autotask_ticket_id`.
- Auto-resolving `alert_tickets` rows by watching linked Autotask ticket status via the existing `ticketUpdate` job.
- Teams channel routing/filtering by alert category (explicitly deferred — separate planned feature handles ticket visibility).

## Verification

1. Run `npm run migrate:up` (from `ai-helpdesk-app/`) and confirm `alert_events`/`alert_tickets` tables + partial unique index exist.
2. `npm run test` — new unit/fixture tests pass alongside existing suite.
3. Manually POST sample payloads (`curl` with `X-Alert-Secret`) to `/api/alerts/backup` twice for the same appliance serial (different device/reason) and confirm: first call creates an `alert_tickets` row and sends a Teams card; second call appends to the same row (no duplicate) and sends an updated card; a third call with a different appliance serial creates a second, independent row.
4. Confirm a request missing/with wrong `X-Alert-Secret` gets 401.
