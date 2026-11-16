# Hosted in-app notification providers for Cadence

Research for [Investigate hosted in-app notification providers for Cadence](https://github.com/johnson-km/cohort-005-project/issues/3), part of [Plan an implementation-ready spec for Cadence in-app notifications](https://github.com/johnson-km/cohort-005-project/issues/1).

Access date: **2026-11-16** (session date). Compared **Knock and Novu Cloud**, one alternative within the agreed maximum of two. Only current first-party documentation, pricing and source were used. This is desk research: no accounts provisioned, SDKs installed, credentials obtained, API deliveries attempted or app implementation changed. Provider selection, events, recipients and UI remain human decisions.

## Findings that affect the decision

Both services supply hosted per-recipient notification storage and state, real-time clients and React components or headless APIs. Neither removes Cadence's responsibility for event correctness, recipient authorization or business actions. [Knock UI][K1], [Novu Inbox][N1]

- **History:** Knock documents indefinite feed availability, separately from expiring debugging logs. Novu's current pricing matrix specifies 30-day Inbox retention on Free and 90 days on Pro/Team. A long-lived activity history has different cost and retention implications from a short-lived attention feed. [Knock retention][K2], [Novu pricing][N2]
- **Mutable work:** Knock explicitly cannot edit a delivered feed item's content/metadata; archive it and send a replacement. Novu documents engagement and action-state changes, but an API to rewrite delivered subject/body was not established here. Do not promise a live task queue merely because either product offers an inbox. [Knock feeds][K3], [Novu notification model][N3]
- **Identity:** Both require additional server-generated proof for private feeds; public identifiers plus guessable numeric IDs alone are insufficient. Knock enhanced security is off by default. Novu requires enabling its HMAC setting. [Knock authentication][K4], [Novu production][N4]
- **Reliability:** Knock trigger idempotency caches successful responses for 24 hours. Novu documents a 24-hour cache including errors, and says idempotency is not enabled for all organizations. Confirm enablement before relying on it; transaction IDs are not equivalent API-boundary protection. [Knock API][K5], [Novu idempotency][N5]
- **Commercial fit:** Knock Free includes 10,000 delivered messages/month, with Starter at $250/month including 50,000. Novu Free includes 10,000 workflow runs/month; Pro starts at $30/month including 30,000. These units differ. Neither headline alone establishes Cadence's bill. [Knock pricing][K6], [Novu pricing][N2]

## Cadence integration context

Repository observations are pinned to baseline [`979e28535d8b0ddd32b7241e51423c9a381879ad`](https://github.com/johnson-km/cohort-005-project/tree/979e28535d8b0ddd32b7241e51423c9a381879ad):

| Observation | Source at baseline | Implication (engineering inference) |
| --- | --- | --- |
| React Router 8.3.0, React/React DOM 19.2.7, TypeScript; SSR enabled | [package.json](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/package.json), [router configuration](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/react-router.config.ts) | Verify actual SDK imports, server render and hydration with these versions. |
| SQLite through better-sqlite3/Drizzle; numeric autoincrement user IDs | [database](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/db/index.ts), [schema](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/db/schema.ts) | Map an existing user to a stable string provider ID. No database replacement is indicated. |
| HTTP-only cookie session, numeric `getCurrentUserId`; development identity switching | [session](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/lib/session.ts), [DevUI](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/components/dev-ui.tsx) | Mint notification credentials from the authenticated server session; reset clients when identity changes. Existing development authentication is not evidence of production hardening. |
| Shared application sidebar and Sonner toaster; no existing inbox/provider dependency found | [application layout](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/routes/layout.app.tsx), [sidebar](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/app/components/sidebar.tsx), package above | Reuse the shell; transient action feedback and persistent notifications have separate jobs. |
| README names Knock for a future integration | [README](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/README.md) | Existing setup guidance is context, not an agreed provider decision. |

Use the existing [glossary](https://github.com/johnson-km/cohort-005-project/blob/979e28535d8b0ddd32b7241e51423c9a381879ad/GLOSSARY.md): Students, Instructors, Platform Admins and Team Admins are distinct audiences. A Team Admin cannot decide Refund Requests.

## Capability comparison

All entries below are documented unless explicitly labelled inference or unverified.

| Concern | Knock | Novu Cloud |
| --- | --- | --- |
| Server-triggered delivery | Node/TypeScript SDK and REST workflow trigger; explicit recipients, optional actor and data. [K7] | `@novu/api` and REST trigger select a workflow and subscribers; direct trigger supports up to 100 recipients. [N6] |
| Secure browser identity | Public API key plus server-signed RS256 user JWT under enhanced security. JWT subject binds the user; expiration and explicit grants can restrict feed read/write and tenant access. [K4] | Public application identifier plus server-generated HMAC-SHA256 subscriber hash, with HMAC enforced in integration settings. Context-scoped sessions also require a context hash. [N4] |
| React / TypeScript / SSR | React components, hooks and JS/TS clients. Current source package metadata includes React/React DOM 19 peers. Exact Cadence SSR compatibility is **unverified**, not guaranteed by peer dependencies. [K1], [K8], [K9] | React 18/19 supported; official Remix quickstart and React Router navigation examples. Exact React Router 8 SSR/hydration compatibility is **unverified**. [N1], [N7], [N8] |
| Packaged versus custom UI | Bell, popover and full-page feed; hooks for a custom UI. [K1] | Composable Bell/Notifications/InboxContent/Preferences, plus headless hooks/JS SDK. [N1], [N9] |
| State and badge semantics | Distinct seen/read/interacted/archived state. Default feed badge counts unseen; displaying the feed marks seen; item click or mark-all marks read. Archive hides an item. Custom behavior is possible. [K10] | Separate `isSeen`, `isRead`, `isArchived`, snooze and action-state methods; headless counts can select unread/unseen/total. Exact chosen UI mark-on-open/click behavior must be specified and verified. [N3], [N9] |
| Updates and pagination | Hosted websocket service; feed client receives updates and fetches subsequent pages. Feed API returns reverse chronological items. [K11], [K12] | `useNotifications` supports filtering, `hasMore`, `fetchMore`, refetch and websocket updates for arrivals/state changes. [N3] |
| Delivered content changes | Explicitly unsupported; archive/reissue. [K3] | Engagement/action changes documented. In-place subject/body replacement **not established**; do not infer support from state-update endpoints. [N3] |
| Trigger retries / duplicate suppression | `Idempotency-Key` is supported specifically for workflow trigger, 255-character maximum, identical parameters required; 2xx responses cached 24 hours, errors not cached. [K5] | API-key authenticated POST/PATCH idempotency, 255-character maximum; success/error cached 24 hours, in-progress lock 5 minutes; concurrent duplicate can return 409. Organization enablement caveat applies. [N5] |
| In-app delivery meaning | Delivered means created in the recipient's feed, not viewed or acted on. [K13] | Inbox messages are stored in Novu and pushed over websocket; no external delivery provider is involved. [N10] |
| Environments | Per-environment data and keys; enable development environment, then promote versioned content. [K14] | Development/Production by default; isolated subscribers, keys, integrations and logs. Workflows published from Development; custom environments require Team/Enterprise. [N11] |
| Debugging | Message delivery/status information plus plan-limited logs; enterprise observability/analytics export. [K13], [K2], [K6] | Activity Feed has per-subscriber runs, step outcomes, payload previews, request details and transaction correlation. [N12] |
| Deletion | Hard user deletion removes properties, preferences, channel data, schedules, subscriptions, messages and related activity; processing may take up to ten minutes. [K15] | Subscriber deletion removes associated messages, preferences and topic subscriptions. Permanent single-notification deletion is also documented. Full backup/log erasure timing **not established**. [N13], [N14] |

**Inference:** these are hosted notification backends with UI embedded in Cadence, not a separate hosted web page to which Students must be sent. Choosing headless transfers rendering, accessibility and interaction work to Cadence; storage and state remain provider-backed. Custom rendering does not establish permission to remove free-tier branding. [K1], [K6], [N1], [N4]

## Actionable notifications and activity updates

These are discussion examples, not selected events or promises that every destination exists.

| Audience | Actionable question to explore | Activity-update question to explore |
| --- | --- | --- |
| Student | Does a response or course task need attention? | Should the Student hear that their Refund Request was decided or a course changed? |
| Instructor | Does course discussion need their response? | Are enrollment or course activity summaries useful? |
| Platform Admin | Which Open Refund Requests need a decision? | Should processing outcomes or failures be surfaced? |
| Team Admin | Is there work managing a Team Licence or Seats? | Should Seat claims or licence changes be reported? |

**Engineering inference:** a notification can link to an existing authorized Cadence page, or a custom handler can submit a Cadence action. Novu explicitly exposes primary/secondary action callbacks; Knock supports action URLs and interaction tracking. These UI events do not approve a Refund Request, grant an Enrollment or verify that a Seat is still available. Cadence must recheck identity, permissions and current resource state server-side. [Novu click behavior][N8], [Knock feed behavior][K10]

A historical activity can truthfully say an event happened even after the resource changes. An actionable item needs a separate policy for completion by someone else, expiry, revocation, deletion and stale links. Keep seen/read separate from work completed. Knock's immutable message content is especially relevant if the desired experience continually rewrites task status. With either provider, clarify whether obsolete items remain historical, are archived, or are replaced before specifying acceptance criteria. This is a design consequence, not a provider selection. [K3], [N3]

## Reliability and security boundary for an implementation design

The following is proposed engineering direction, **not an approved implementation plan**:

1. A successful, authorized Cadence mutation records the domain change and a durable notification intent together in SQLite. Dispatch afterward with a stable event/recipient key and an immutable payload. A database transaction cannot atomically commit a remote HTTP request; an outbox/worker or equivalent durable mechanism closes the crash gap. Neither provider's trigger deduplication replaces this local responsibility.
2. Resolve recipients on the server from Cadence relationships. Send the minimum necessary provider profile and payload. IDs should be stable strings, for example `cadence-user:<numeric-id>`, with environment/data-generation separation so reseeding development user IDs cannot expose old notifications to a different person. This mapping is a proposal, not a required provider format.
3. Keep backend API keys and signing secrets server-only. Return only the authenticated user's client proof, public identifiers and permitted scope. Do not accept an arbitrary browser user ID to mint proof. Logout/DevUI switching must disconnect sockets and clear cached data. For Knock, choose short JWT lifetime and a refresh path. For Novu, the documented subscriber hash has no expiry input; per-session revocation/rotation behavior remains to be verified. [K4], [N4]
4. Persist attempt count, last error, next attempt, domain event ID, recipient ID, provider workflow/run/request/message correlation and dispatch state. Retry transient network/rate-limit failures with bounded backoff. Escalate permanent configuration/authorization failures. Provider HTTP acknowledgment, feed storage, browser display and business completion are different milestones. [K13], [N12]
5. Respect each provider's duplicate window. Knock retries after failures execute again; Novu may replay a cached error for 24 hours. Define reconciliation and manual recovery for unknown outcomes and attempts older than the cache window; do not silently change an idempotency key to force a repeat. [K5], [N5]
6. Keep new workflows in-app only and existing email paths unchanged. Provider examples that add email fallback are outside this map. Assert the configured workflow has no new email, SMS or push step.

Neither service's documentation establishes a Cadence-specific end-to-end exactly-once or latency guarantee. Novu's retry policy distinguishes framework callbacks, webhooks and channel sends; its in-app path has no external provider retry. Do not import an email retry policy into this design. [N10]

## Retention, data handling and location

Knock logs expire after 30 days on non-Enterprise plans, 90 days on Enterprise; API logs have a universal 30-day window. Feed items remain available indefinitely even after related logs expire. Its pricing says feed retention can be configured, while retention documentation says custom retention policies are not currently supported: confirm the exact supported feed-cleanup control before specifying a custom TTL. [K2], [K6]

Novu's pricing separates **Activity Feed** debugging retention (24 hours Free, seven days Pro) from **Inbox feed** retention (30 days Free, 90 days Pro/Team). These are not interchangeable. Its current Free offering advertises US/EU residency. Knock's account documentation describes EU residency as coming by end-2026, not available now; confirm availability rather than treating a roadmap date as delivered. [N2], [K16]

**Proposed data policy:** provider payloads contain event type, stable resource references, safe display text and an allowed internal destination. Keep refund reasons, payment details, sensitive discussion content, session cookies and secrets out unless a specific requirement justifies transmission. An ID-linked notification can still disclose a person's activity. Route authorization prevents future page access but cannot retract sensitive text already delivered to a browser.

Define required history duration, notification deletion when account/resource access changes, profile deletion propagation and recoverable dispatch-data retention. Provider user deletion endpoints establish API capabilities, not a complete Cadence retention policy or backup-erasure SLA. Confirm region, DPA/subprocessors and deletion/backups with the chosen service if those become requirements; no legal/compliance approval is implied. [K15], [N13]

## Pricing and practical constraints

Prices are USD as displayed on access date, not a quote. Recheck before provisioning or purchase.

| Item | Knock | Novu Cloud |
| --- | --- | --- |
| Free monthly allowance | 10,000 delivered messages; unlimited stored profiles/workflows and isolated environments | 10,000 workflow runs; unlimited subscribers, 20 workflows, two environments, up to three team members |
| First paid plan | Starter $250/month; 50,000 messages, then $0.005/message | Pro from $30/month; 30,000 runs, additional $1.20/1,000 |
| UI branding | Removal starts on Starter | Removal on paid plans |
| Longer user-visible history | Feed retention listed unlimited on all plans | Free 30 days; Pro/Team 90 days; Enterprise custom |

All cells: [Knock pricing][K6], [Novu pricing][N2], [Novu production][N4]. Knock's FAQ says it does not hard-cap messages without a billing method and will contact the customer after exceeding Free; do not model Free as a spend-control mechanism. Novu Free over-limit behavior and exact applicable paid-volume selection were not independently exercised. [K6]

Estimate **recipient fan-out**, not just domain-event count. Novu documents one workflow run per matching topic subscriber; one run can generate multiple messages. For a deliberately simple hypothetical with 5,000 recipient deliveries/month, one run and one in-app item each, both headline free allowances cover volume, subject to other constraints. Digests, skipped steps, multiple items and broad audiences change that comparison. This example selects no event set or audience. [Novu usage][N15], [K6], [N2]

## Verification still needed with a real service

No account or payment is needed to use these findings in the next human decision. Once a provider is selected and credentials are available, make these checks explicit in the implementation acceptance plan:

- Actual React Router SSR build and hydration with a pinned SDK version, CSS and client initialization; anonymous rendering, loading/error/offline states, keyboard/focus behavior and narrow-screen sidebar/popover fit.
- Two authenticated users: isolated inboxes, counts and mutations; forged/missing proof rejected; server refuses token issuance for another user. Include two Team Admins if team scoping is selected.
- Server-trigger one real in-app notification in an isolated development environment; observe receipt while open, after reconnect and after reload. Novu keyless mode uses temporary demonstration data and does not establish this. [N16]
- Confirm seen/read/badge/mark-all/archive behavior, pagination and cross-tab state. Do not assume automatic state convergence under disconnection without testing.
- Trigger timeout/retry with the same key, verify one intended run/item; for Novu verify organization idempotency enablement and cached-error recovery. Exercise crash-after-domain-commit recovery and expired deduplication windows. [K5], [N5]
- Change identity through DevUI and logout/login; ensure old items/sockets disappear. Missing session should never fall back to a shared subscriber.
- Verify an expired, deleted or no-longer-authorized destination and completion elsewhere; decide historical versus active-work behavior.
- Inspect correlated app/provider logs without secrets; demonstrate operator recovery from a configuration failure. Check environment keys and production publication deliberately.
- Verify requested deletion and history/retention behavior; inspect actual account usage/branding terms. No new outbound channel is configured and existing emails behave as before.

## Remaining decisions and research limits

The existing human purpose/events and UI decisions remain open. Choose actionable notifications, activity updates or an intentional combination; choose audiences, recipient rules, history duration, stale-item behavior and badge semantics before final provider selection. Weight cost, region, branding, environment count and mutable-work needs against those choices.

Specific unresolved facts are actual React Router 8 compatibility, Novu idempotency enablement for the eventual account, Novu delivered-content editing, credential revocation details, configurable Knock feed TTL, exact account commercial terms and data-erasure timing beyond the documented APIs. No unsupported capability was counted as absent. These can be resolved during provider-specific verification once a human has narrowed requirements; this research does not justify choosing a provider or provisioning infrastructure on their behalf.

## Primary sources

Every source below was accessed 2026-11-16. Documentation and pricing are mutable; this file records what was retrieved, not a contractual guarantee.

[K1]: https://docs.knock.app/in-app-ui/react/overview "Knock: React UI components and headless hooks"
[K2]: https://docs.knock.app/manage-your-account/data-retention "Knock: Data retention"
[K3]: https://docs.knock.app/in-app-ui/feeds/overview "Knock: Feeds and immutable delivered content"
[K4]: https://docs.knock.app/in-app-ui/security-and-authentication "Knock: Client security, JWT and grants"
[K5]: https://docs.knock.app/api-reference/overview/openapi "Knock: API idempotency"
[K6]: https://knock.app/pricing "Knock: Current pricing and feature comparison"
[K7]: https://docs.knock.app/send-notifications/triggering-workflows/api "Knock: Server workflow triggers"
[K8]: https://docs.knock.app/developer-tools/sdks "Knock: Server and client SDKs"
[K9]: https://github.com/knocklabs/javascript/blob/main/packages/react/package.json "Knock: React package metadata (moving main branch)"
[K10]: https://docs.knock.app/integrations/in-app/feeds "Knock: Default feed status and interaction behavior"
[K11]: https://docs.knock.app/in-app-ui/api-overview "Knock: Feed API and real-time service"
[K12]: https://docs.knock.app/in-app-ui/javascript/sdk/feed-client "Knock: Feed client and pagination"
[K13]: https://docs.knock.app/send-notifications/message-statuses "Knock: Delivery and engagement statuses"
[K14]: https://docs.knock.app/version-control/environments "Knock: Isolated environments"
[K15]: https://docs.knock.app/managing-recipients/deleting-users "Knock: Hard deletion and processing time"
[K16]: https://docs.knock.app/manage-your-account/knock-plans "Knock: Plan details, residency roadmap and in-app sandbox caveat"
[N1]: https://docs.novu.co/platform/inbox "Novu: Inbox components, React versions and customization"
[N2]: https://www.novu.co/pricing/ "Novu: Current pricing and retention matrix"
[N3]: https://docs.novu.co/platform/sdks/react/hooks/use-notifications "Novu: Notification types, actions, pagination and updates"
[N4]: https://docs.novu.co/platform/inbox/prepare-for-production "Novu: HMAC identity, context and branding"
[N5]: https://docs.novu.co/api-reference/idempotency "Novu: Idempotency enablement, TTLs and transaction-ID distinction"
[N6]: https://docs.novu.co/api-reference/events/trigger-event "Novu: Trigger event API and recipient limit"
[N7]: https://docs.novu.co/platform/quickstart/remix "Novu: Remix integration quickstart"
[N8]: https://docs.novu.co/platform/inbox/advanced-customization/notification-click-behavior "Novu: Router and action callbacks"
[N9]: https://docs.novu.co/platform/inbox/headless-mode "Novu: Headless hooks and counts"
[N10]: https://docs.novu.co/platform/developer/delivery-retries "Novu: Delivery retry policy and in-app path"
[N11]: https://docs.novu.co/platform/developer/environments "Novu: Environment isolation and publication"
[N12]: https://docs.novu.co/platform/workflow/monitor-and-debug-workflow "Novu: Workflow and request observability"
[N13]: https://docs.novu.co/api-reference/subscribers/delete-a-subscriber "Novu: Subscriber deletion"
[N14]: https://docs.novu.co/api-reference/subscribers/delete-a-notification "Novu: Permanent notification deletion"
[N15]: https://docs.novu.co/platform/workflow/usage "Novu: Runs, messages and fan-out usage semantics"
[N16]: https://docs.novu.co/platform/inbox/setup-inbox "Novu: Real subscribers versus keyless demonstration data"
