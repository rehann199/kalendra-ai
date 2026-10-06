# Technical Case Study

Kalendra combines three hard problems in one consumer app: multi-provider calendar IO, a real-time assistant channel, and calendar visualisation that stays honest about overlapping time ranges. The implementation keeps provider tokens and layout logic on the client while pushing interpretation, provider API calls and usage enforcement to an app backend the mobile client already trusts for sign-in lifecycle.

## Calendar integration

### Connection model

Each connected calendar is stored locally with provider type (Google or Microsoft), email address, display metadata, colour, time zone and OAuth tokens with expiry timestamps. Connecting launches PKCE OAuth in an external browser; returning URIs are classified by scheme and host to determine which provider completed.

Before any fetch or assistant turn, the auth layer walks all accounts and refreshes access tokens when expiry is near. Refresh failures that indicate a revoked grant mark the account as needing reconnect rather than sending stale tokens indefinitely.

### Unified reads

The calendar home screen computes a fetch window from the visible month anchor: roughly the calendar month plus padding days so week edges still have context. It calls the backend with the fresh account list, a minimum and maximum instant, and a result cap. Successful responses decode into a shared event model that carries calendar identity, colour, invitation response state and timing fields used across tabs.

Tasks load through a sibling service with their own fetch and mutation helpers; the schedule tab merges tasks with timed events for checklist UX.

### Manual mutations

Event detail actions (RSVP, edit fields, add or remove guests, cancel) and task create/update/complete paths all resolve the owning account from the event, build a provider type discriminator, and call backend update helpers. Guest list edits prefer server-side merge semantics for removals so the client does not drop attendees it never downloaded.

Manual create sheets gather title, time, recurrence, reminders, guests and calendar picker, then post through the same backend create APIs the assistant eventually uses after confirmation.

## Assistant boundary

### Turn transport

Text turns open a long-lived HTTP client with extended idle timeout because the backend may run speech transcription and language-model work before the first streamed byte. The client sends multipart form fields for message text and JSON-encoded conversation history, plus headers for identity, connected accounts, build metadata, subscription context and device local time zone.

The response is parsed as server-sent events. Besides errors, the stream can yield a final response payload, optional card type and card body for UI tiles, audio chunks for playback, usage counters, rate-limit signals, session-expired signals and email-verification requirements.

Voice turns upload recorded bytes instead of text but share the parser and downstream UI.

### Proposal versus execution

When a turn includes a pending action structure, the chat layer stops short of calling execute immediately. Create and update actions navigate to manual sheets with fields filled from the proposal; only after the user saves does the backend count the change. Delete and certain task operations show a reusable confirm sheet; execute runs with an explicit confirmed flag and the target account email embedded in the action map.

A separate execute path also exists for mutations that already happened inside an editor but still need to count against monthly action limits.

Session history lives only in memory on the device until cleared; clearing chat wipes the list locally because the backend treats each request as stateless aside from the history bytes sent along.

## Conflict layout engine

Conflict detection operates on the in-memory event list for a day. Events sort into groups where every member mutually intersects in time. Groups classify as none, partial overlap or exact match on start and end instants.

Rendering differs by tab. Day view can place overlapping cards side by side with tinted bands filling the shared interval, dashed separators for exact conflicts spanning providers, and tappable badges that open the overlap sheet. Week view compresses badges for narrow columns. Month view truncates titles into chips and may stack multiple chips in a cell.

The overlap sheet title switches between partial and full wording; full conflicts repeat the shared time range in the footer. Selecting a row opens the standard event detail sheet with sibling metadata passed through so banners stay consistent.

Automated widget tests cover grid layout, scroll-to-now behaviour and confirm sheets, signalling that conflict rendering is treated as regression-sensitive UI logic.

## Sync and freshness

Kalendra is not an offline-first replica of every calendar row on disk. Instead:

- **Account metadata** syncs to the backend when users rename calendars, change colours or adjust time zones, and can be refreshed from the backend when preference screens open.
- **Events** refresh when the user changes period, returns to the calendar tab, or completes a mutation callback.
- **Assistant context** always sends the current account list and tokens on the next turn so reconnecting in settings propagates without restarting the app.

Push notification registration posts device tokens to the backend for remote messaging features. Local notifications schedule daily brief copy from fixed templates at the user-selected time, with background work to refresh scheduled notifications when settings change.

## Monetisation and limits

Subscriptions integrate through the cross-store in-app purchase plugin. The backend enforces plan-specific daily conversation allowances and monthly scheduling-action counts; the client surfaces remaining usage in chat and intercepts limit exceptions with upgrade sheets. Exact numeric limits are product tuning and are not published here.

## Testing footprint

The production tree includes thirteen automated test files spanning conflict widgets, confirm sheets, greeting copy, preferred-hours calculations, onboarding layout overflow and daily brief text selection. This portfolio does not claim continuous integration status or that all tests pass in every environment.

## Scope note

Backend implementation details, language-model prompts and provider API credentials live outside this repository. The mobile client code reviewed for this write-up is the source of behavioural claims above.
