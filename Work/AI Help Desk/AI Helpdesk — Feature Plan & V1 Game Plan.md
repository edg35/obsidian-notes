Planning breakdown for the AI-powered Teams service desk assistant. Each item below is scoped to become a ticket. Tags: **[V1]** = required for a complete first version, **[V1-secondary]** = in scope for V1 but can be cut/deferred if the timeline tightens, **[Post-V1]** = backlog.

Grounded against the current codebase (`ai-helpdesk-app/`, branch `llm-integration`) as described in `CODEBASE_OVERVIEW.md`.

---

## The V1 target loop

This is the flow every V1 ticket serves. Keep it in mind when writing acceptance criteria.

1. A ticket comes in / goes stale → the bot proactively posts an **alert** about it (this already exists as one-way Adaptive Cards from the cron jobs).
2. A tech responds — either in a **shared Teams channel** or in the **1:1 app chat** — by **replying-with-quote** to the bot's message about that ticket (free-text like _"Took care of it. Reboot fixed it."_).
3. The bot **correlates that reply to exactly one ticket** using the quoted message.
4. The **LLM decides** what to do: draft a comment, close the ticket, or ask the tech for clarification.
5. The bot shows the tech the **proposed change** with **Approve / Modify / Cancel** buttons.
6. On **Approve**, the bot **writes to Autotask** (adds the note and/or updates status). On **Modify**, the tech edits it. On **Cancel**, nothing happens.

**Non-negotiable invariant (from the meeting):** every bot message is about **one ticket only**, and free-text ticket updates **require a reply-with-quote** so the target ticket is never ambiguous.

---

## Where the current code stands (the gaps V1 must close)

- **No correlation model.** `conversations`/`conversation_messages` track threads and log messages, but nothing links a message or card to a ticket or a pending action. This is the central missing piece.
- **Inbound is a stub.** `teamsApp.on('message')` only echoes `You said: …`. No parsing, no reply detection, no LLM in the loop.
- **Cards are one-way.** Only `Action.OpenUrl` is used. No `Action.Submit` / `card.action` handler exists (the SDK supports it).
- **No Autotask write path.** Everything is read-only queries. Adding a note or closing a ticket must be built from scratch.
- **`llmService` has no callers** and no prompt library.
- **No alert category concept** (Performance / Downtime / Critical Services) anywhere.
- **Conversation references are in-memory** (`LocalStorage`) and may not survive a restart — risk for proactive sends.
- **Card-building is duplicated** across two job files; a third copy is coming unless refactored.

---

## Epic A — Foundation: correlation & shared state

The prerequisites everything else depends on. Build these first.

- **A1 [V1] — Pending-action / message↔ticket correlation model.** New table (e.g. `pending_actions`) plus a proper TS type. Records: the outbound Teams message/card, its `teams_message_id`, the `ticket_id` it concerns, the conversation it was sent to, the action type, a state (`pending` → `applied` / `cancelled` / `expired`), the LLM draft, and timestamps. This is what lets a later reply or button click resolve back to one ticket + one intended change. _Depends on: none. Blocks: A2, B2, D1–D4. Note: include `org_id` from the start — see J1._
    
- **A2 [V1] — Tag outbound messages/cards with their ticket.** When an alert card is sent, capture the returned Teams message id and persist the message↔ticket link (via A1). Today `sendCardToConversation` logs the card but not a correlation key. _Depends on: A1._
    
- **A3 [V1] — Durable conversation-reference storage.** Replace the SDK's default in-memory `LocalStorage` with a persistent store so proactive escalation sends survive a process restart. **Start with a short spike** to confirm whether `teamsApp.send(conversationId, …)` can reconstruct a reference from `conversationId` alone or genuinely needs a cached reference (Open Question #1 in the overview). Then implement accordingly. _Depends on: none. Note: could be [V1-secondary] if restarts are rare and users re-message the bot, but it's cheap insurance._
    
- **A4 [V1] — Shared Adaptive Card builder module.** Factor the near-identical `buildTicketAlertCard` out of `newTicketNotification.ts` and `automatedTicketNotification.ts` into one module before adding the new confirmation card. Prevents a third copy. _Depends on: none. Blocks (cleanly): D2._
    

---

## Epic B — Inbound handling & reply correlation

- **B1 [V1] — Replace the echo handler with a real inbound router.** Rework `teamsApp.on('message')` to classify inbound activities and route them (quoted reply vs. plain message vs. unrecognized) instead of echoing. _Depends on: A1._
    
- **B2 [V1] — Reply-with-quote detection → ticket lookup.** Read `activity.replyToId` (and/or the quoted-message attachment) on inbound messages, look up the referenced outbound message via A1/A2, and resolve the single ticket it maps to. _Depends on: A1, A2, B1._
    
- **B3 [V1] — Enforce the quote-reply requirement.** If a tech sends free-text that isn't a quote-reply to a bot ticket message, don't guess — respond with a short nudge telling them to reply-with-quote to the relevant alert. This is the meeting's explicit rule and the disambiguation guarantee. _Depends on: B2._
    
- **B4 [V1] — Work in both channel and 1:1 (app) scopes.** Confirm correlation and sending work when the tech replies in a shared Teams channel **and** in the personal app chat. Channel vs. personal scope changes activity shape and how proactive sends address the conversation. _Depends on: B2._
    
- **B5 [Post-V1] — Free-text without quote (intent matching).** Let a tech respond naturally and have the bot infer the ticket from open/assigned tickets. Deliberately **out of V1** — the quote requirement replaces it for now.
    

---

## Epic C — LLM decision & drafting

- **C1 [V1] — LLM decision service.** Given ticket context + the tech's reply, return a **structured JSON decision**: action ∈ {`comment`, `close`, `request_clarification`} plus the drafted comment/resolution text (polished from shorthand — e.g. _"Took care of it. Reboot fixed it."_ → a professional closing note). Build on `callLlmJson` in `llmService.ts`. _Depends on: B2 (needs the ticket + reply). Blocks: D2._
    
- **C2 [V1] — Prompt templates + output schema + validation.** System prompt(s) for the decision/drafting task, a defined JSON schema, and runtime validation of the model's output (reject/repair malformed responses). No prompt library exists yet. _Depends on: C1 (built together)._
    
- **C3 [V1] — Ticket context assembly.** Helper that gathers what the LLM needs — ticket fields, recent notes (`getTicketNotes`), status — into a compact context payload. Keeps token use sane and decisions grounded. _Depends on: none (uses existing Autotask read calls)._
    
- **C4 [Post-V1] — Trend / common-alert identification.** Aggregate across tickets to surface recurring alerts and patterns. Backlog.
    

---

## Epic D — Confirmation flow (Approve / Modify / Cancel)

- **D1 [V1] — Wire up card action handling.** Register a `card.action` / `Action.Submit` handler (the SDK supports it; nothing uses it today). Each button carries a `data` payload with the pending-action id so a click resolves unambiguously to one ticket + one change. _Depends on: A1._
    
- **D2 [V1] — Confirmation card.** Build a card (via the A4 shared builder) that shows the LLM's proposed change and renders **Approve / Modify / Cancel** buttons. Sent to the tech after C1 produces a draft. _Depends on: A4, C1, D1._
    
- **D3 [V1] — Modify sub-flow.** Let the tech edit the draft — either an inline `Input.Text` on the card or a follow-up prompt — then re-confirm before applying. Decide the UX here. _Depends on: D2._
    
- **D4 [V1] — Idempotency & state machine.** Enforce the `pending → applied/cancelled/expired` lifecycle on the pending action so a change can't be double-applied (e.g., button clicked twice, or reply + button). Guard the Autotask write behind this state. _Depends on: A1, E1/E2._
    
- **D5 [V1-secondary] — Authorization.** Decide who may approve a change (any channel member vs. assigned tech only) and add at least a minimal guard. Flag for a product decision; a permissive V1 with logging may be acceptable short-term. _Depends on: D1._
    

---

## Epic E — Autotask write-back

- **E1 [V1] — Add a note to a ticket.** Net-new mutation: `POST TicketNotes` to attach the approved comment/resolution. No write path exists in `autotaskApiCalls.ts` today. _Depends on: none (structurally). Blocks: D4._
    
- **E2 [V1] — Update ticket status / close.** Net-new: `PATCH Tickets/{id}` to move status (e.g., to Resolved/Complete) when the decision is `close`. _Depends on: E4._
    
- **E3 [V1] — Write-failure handling.** On Autotask write failure, surface it back into the Teams thread and leave the pending action re-tryable rather than silently swallowing (current pattern is `console.log` + continue). _Depends on: E1, E2._
    
- **E4 [V1] — Fix / validate `AUTOTASK_REGION`.** The local `.env` is missing `AUTOTASK_REGION`, which yields a malformed Autotask URL with no hard error. Add explicit validation at startup. This is small but a correctness blocker for every write call. _Depends on: none. Do early._
    

---

## Epic F — Alert categorization (Performance / Downtime / Critical Services)

Led the meeting, but largely separable from the core loop. Recommend treating as **V1-secondary**: ship the conversational loop first, then layer categories.

- **F1 [V1-secondary] — Category concept.** Add a category field/table + TS type with an initial set: Performance, Downtime, Critical Services (names still debatable per the meeting). _Depends on: none._
    
- **F2 [V1-secondary] — Assignment logic.** Derive a ticket's category — via keyword rules on title/description, an LLM classification pass, and/or a queue→category mapping. Note: there's no external alert feed yet, so categorization runs on Autotask tickets for now. _Depends on: F1._
    
- **F3 [V1-secondary] — Surface categories in alerts.** Label the category on the alert card and/or route categories to separate channels. _Depends on: F1, F2, A4._
    

---

## Epic G — Attachments & documentation (future)

- **G1 [Post-V1] — Accept URL/attachment on a resolution.** Let a tech include a link or file with their resolution so it's saved on the ticket. No attachment handling exists today (`activity` handler only reads `.text`).
    
- **G2 [Post-V1] — IT Glue write-back.** Client + integration to push resolutions into IT Glue documentation. Fully greenfield (no IT Glue client exists).
    

---

## Epic H — Additional alert sources (future)

- **H1 [Post-V1] — Ingestion framework.** A reusable webhook/polling pattern for external alert feeds (none exists — today everything is derived from Autotask ticket status).
    
- **H2 [Post-V1] — Source integrations.** Backups (Datto Endpoint Backup, Azure backup, DropSuite, Veeam/FSAI, Wasabi), network health (Auvik, Ubiquiti, SonicWall, Meraki, Netgear), email health (KnowBe4, Inky, Defender→EXO), endpoint security (Defender/Huntress), Synology monitoring, critical servers/services.
    

---

## Epic I — Cross-cutting hardening

- **I1 [V1] — Observability on the loop.** Minimal structured logging/metrics across the correlate→decide→confirm→write path so failures in the new loop are visible (current handling is sparse `console.*`).
    
- **I2 [Post-V1] — Test coverage.** Only `llmService.test.ts` exists. Add coverage for the correlation model, inbound router, card handler, and Autotask writes.
    

---

# Phase 2 — Multi-tenant web portal

The Phase 1 loop above proves the product with **your own MSP as the first tenant**. Phase 2 turns it into something other companies log into and configure themselves: sign in with a Microsoft account, get walked through onboarding, and run their company's configuration from a portal.

**The one thing to pull forward into V1:** make the data model multi-tenant _now_. Adding an `org_id` column while there's exactly one org is nearly free; retrofitting tenant scoping onto live tables full of data later is the expensive, error-prone path. So J1/J2 below are tagged **[V1]** even though the rest of the portal is **[Phase 2]**. Everything else — auth, portal UI, onboarding — can follow once the loop works.

Tags in this phase: **[V1]** = pull forward into the first version, **[Phase 2]** = the portal build itself, **[Post-V1]** = later.

---

## Epic J — Tenancy foundation

- **J1 [V1] — `organizations` table + `org_id` on every tenant-scoped table.** Add an org concept and stamp `org_id` on `tickets`, `conversations`, `conversation_messages`, `pending_actions`, the picklist lookup tables, and anything else per-company — even though only one org (yours) exists at first. This is the cheap-now / painful-later decision. _Depends on: none. Do before/with A1._
    
- **J2 [V1] — Enforce org scoping on all data access.** Every query must be constrained to one org so nothing can leak across tenants. Decide the mechanism: a query helper/repository layer that always injects `org_id`, or Postgres **Row-Level Security**. RLS is stronger but changes how connections are set up. _Depends on: J1._
    
- **J3 [Phase 2] — Runtime tenant resolution.** Map an inbound Teams activity to its org (via the Teams/AAD tenant id) and map each Autotask instance to its org, so the bot and jobs act as the correct tenant. Today the jobs read one global config. _Depends on: J1, K1._
    
- **J4 [Phase 2] — Per-org job fan-out.** `jobScheduler` should iterate active orgs and run each escalation/sync job with that org's own config, instead of the current global env-var-driven single run. _Depends on: J3, K1._
    

---

## Epic K — Per-tenant configuration store

- **K1 [Phase 2] — Move config from `.env` to per-org DB config.** Autotask connection settings, queue filters (`EXCLUDED_QUEUES` etc.), business hours, alert categories, and alert routing become per-org rows instead of process-wide env vars. Shared infra (the bot host, the DB) stays env-driven; per-company settings move to the DB. _Depends on: J1._
    
- **K2 [Phase 2] — Encrypt tenant secrets at rest.** Each org's Autotask API secret (and any Teams secrets) must be encrypted — Azure Key Vault references or column-level encryption, never plaintext columns. The portal collects them; the app resolves them at use time. _Depends on: K1._
    
- **K3 [Phase 2] — Connection test during onboarding.** A "Test connection" action that makes a live Autotask (and Teams) call to validate an org's credentials before activating them. _Depends on: K1, K2._
    
- **K4 [decision] — Shared vs. per-tenant Azure OpenAI.** Decide whether the LLM is a product-owned resource with per-org usage tracking, or bring-your-own per tenant. Affects K1's schema and cost model.
    

---

## Epic L — Portal authentication & access

- **L1 [Phase 2] — Microsoft sign-in.** OIDC login with Entra ID (MSAL). Since your customers are Microsoft shops, "sign in with Microsoft" is the natural front door. **Option:** Clerk is already in your connectors and supports Microsoft OAuth — worth weighing against native MSAL (see decisions). _Depends on: none._
    
- **L2 [Phase 2] — User↔org membership + roles.** A signed-in user resolves to their org(s) and a role (admin / member). Admins configure; members may just view. _Depends on: L1, J1._
    
- **L3 [Phase 2] — Protect the API surface.** The existing `/api/*` Express routes are **currently unauthenticated**. Gate them behind portal auth + org scoping before anything is exposed externally. Security blocker for going multi-tenant. _Depends on: L1, J2._
    
- **L4 [Post-V1] — Invite & manage users within an org.**
    

---

## Epic M — Portal & onboarding

- **M1 [Phase 2] — Portal scaffold (frontend + API).** Stand up the portal app. **Decision:** extend the current Node/Express app with a React/Next front end, or build a separate ASP.NET Core API + front end. Either way it shares the same Postgres. (Your AuthHR experience maps cleanly to the ASP.NET Core option; keeping it in Node keeps the stack singular.) _Depends on: L1._
    
- **M2 [Phase 2] — Onboarding wizard.** Guided flow: sign in → create org → connect Autotask → connect Teams → set queues / business hours / categories → run the connection test → activate. This is the "put them through an onboarding process" piece. _Depends on: M1, K1, K3, N3._
    
- **M3 [Phase 2] — Configuration management screens.** Post-onboarding editing of everything M2 collected — the "run all the company configurations from there" surface. _Depends on: M1, K1._
    
- **M4 [Post-V1] — Activity dashboard.** Per-tenant loop metrics, ticket/alert volume, health. The `/api/*` routes already read like scaffolding for exactly this.
    
- **M5 [Post-V1] — Billing / plans / subscription management.**
    

---

## Epic N — Bot & Teams multi-tenancy

- **N1 [Phase 2] — Multi-tenant bot registration.** Configure the Entra bot app as multi-tenant so one bot serves many client tenants, and map each Teams tenant to an org (pairs with J3). Confirm whether one shared registration or per-client registration is the model. _Depends on: J3._
    
- **N2 [Phase 2] — Per-org proactive send addressing.** Proactive escalations must go out with the right service URL / credentials / conversation reference for each tenant. Extends A3 (durable conversation refs) into a per-org store. _Depends on: A3, J3._
    
- **N3 [Phase 2] — Teams provisioning during onboarding.** Admin consent + installing the Teams app into the client's tenant as part of M2. _Depends on: N1._
    

---

## Open decisions (Phase 2)

- **Phasing** — confirm the recommendation: pull only the tenancy _schema_ (J1/J2) into V1, defer the portal. Or do you need to onboard external companies sooner, changing the priority?
- **Auth provider** — native MSAL/Entra vs. Clerk (already connected). Trade-off: control/no-dependency vs. speed.
- **Portal stack** — extend Node/Express + React/Next vs. a separate ASP.NET Core API.
- **Tenant isolation** — Postgres Row-Level Security vs. application-layer scoping (J2).
- **Bot model** — single multi-tenant bot registration vs. per-client (N1).
- **Azure OpenAI** — shared product resource vs. per-tenant (K4).

---

## V1 Definition of Done

The loop works end to end for a real ticket:

- [ ] Bot's ticket alert is stored with a message↔ticket link (A1, A2).
- [ ] A tech's reply-with-quote — in a channel **and** in the app — resolves to the one correct ticket (B2, B4).
- [ ] Non-quoted free-text is rejected with a nudge (B3).
- [ ] The LLM turns the reply into a structured decision + polished draft (C1–C3).
- [ ] The tech sees Approve / Modify / Cancel and can edit (D2, D3).
- [ ] Approve writes the note and/or closes the ticket in Autotask, exactly once (E1, E2, D4).
- [ ] Failures surface back into Teams instead of vanishing (E3, I1).
- [ ] `AUTOTASK_REGION` is validated at startup (E4).

---

## Suggested build order (critical path)

1. **E4** (config fix) + **A4** (card refactor) — quick, unblock cleanly.
2. **A1 → A2** (correlation model) — the spine.
3. **A3** (durable conv. refs) — spike, then implement.
4. **B1 → B2 → B3 → B4** (inbound + reply correlation).
5. **C3 → C1 → C2** (LLM decision).
6. **E1 / E2 → E3** (Autotask writes).
7. **D1 → D2 → D3 → D4 → D5** (confirmation flow) — ties it together.
8. **I1** throughout.
9. Then **F** (categories), then G/H/C4 backlog.

Epics A, C, and E can be developed in parallel by different devs once A1's table shape is agreed; D depends on all three converging.

**Phase 2 sequencing.** Do **J1 + J2 inside Phase 1** — bake `org_id` and org scoping in while there's one org and nothing to migrate. Then, after the loop proves out: **L** (auth) and **K** (config store) come before **M** (the portal needs both), **J3/J4 + N** are what actually let multiple live tenants run at once, and **K2** (secret encryption) must land before any real client credentials are stored.

---

## Open decisions to settle before ticketing

- **Category names** — Performance / Downtime / Critical Services, or others? (meeting flagged as debatable)
- **Modify UX** — inline card input vs. follow-up message?
- **Approval authorization** — anyone in channel vs. assigned tech only? (D5)
- **Is categorization (Epic F) in V1 or the first fast-follow?** — my recommendation is fast-follow, but it was the meeting's headline, so confirm.
- **`EXCLUDED_QUEUES` vs `AUTOMATED_TICKET_NOTIFICATION_QUEUES` overlap** (overview Open Question #5) — relevant if category logic keys off queues.