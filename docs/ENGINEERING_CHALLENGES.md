# Engineering Challenges

Each entry follows the same shape: the problem, the approach taken, the trade-offs, and the outcome. Identifiers from production are generalised on purpose.

## 1. One timeline from two calendar providers

**Problem.** Users expect Google work and Outlook personal calendars in a single scrollable view, but OAuth, token refresh and API quirks differ per provider.

**Approach.** The client stores a typed account record per connection and refreshes tokens centrally before reads. All grid data comes from one backend "unified events" call that accepts every connected account, a time window and a cap. Parsed events carry enough metadata to colour pills, filter hidden calendars and choose the correct provider discriminator on writes.

**Trade-offs.** The phone does not keep a durable full mirror of every event; scrolling to a new month triggers another fetch. Backend availability is on the critical path for viewing, not just assistant features.

**Outcome.** Day, week, month and schedule tabs share one list implementation and one loading/error state machine.

## 2. Streaming assistant turns without fragile HTTP clients

**Problem.** Voice and text turns can spend a long time in transcription and language-model work before bytes return. Default short idle timeouts on mobile HTTP clients abort those requests with generic network errors.

**Approach.** Streaming calls use a dedicated client with multi-minute idle timeout and explicit close after the stream drains. The parser buffers multiline server-sent event data before JSON decode. Chat UI completes its future when the primary response event arrives while continuing to read audio chunks in the background so narration cannot race ahead of visible text.

**Trade-offs.** Holding a connection open consumes battery and ties up a socket during slow turns; the alternative would be polling or push, which the product does not use for chat.

**Outcome.** Voice turns reach the same backend path as typed messages without silent client-side aborts.

## 3. Hard separation between suggestion and calendar writes

**Problem.** Letting the language model write directly to provider APIs would create support burden and trust issues when a proposal misheard a time or account.

**Approach.** Backend turns return structured pending actions. The client routes creates and updates through manual sheets pre-filled from the proposal so the person can edit every field. Deletes and selected task operations require an explicit confirm sheet before the execute call runs with a confirmed flag set. Editors can flag certain saves to count as assistant-class actions for plan limits even when the user typed tweaks.

**Trade-offs.** Extra taps versus faster automation; the product chooses safety and familiarity with standard calendar forms.

**Outcome.** Assistant cards can celebrate a created meeting while the actual write still went through the same validation paths as manual entry.

## 4. Conflict clusters that mean two different things

**Problem.** Calendar UIs often paint all overlaps red, even when one meeting simply runs long into another versus two bookings occupying identical slots.

**Approach.** Group events on a day by mutual overlap. Classify clusters as partial or exact by comparing start and end instants. Render partial clusters with orange badges, tinted overlap bands and "Partial Overlap" copy; exact clusters use red, dashed dividers between side-by-side cards when calendars differ, and footers in the picker sheet that restate the shared slot. Event detail sheets accept sibling lists to drive banner severity.

**Trade-offs.** More layout branches and test surface; week and month tabs simplify visuals compared to day view.

**Outcome.** Widget tests lock day-grid conflict rendering and overlap pickers; users get vocabulary that matches the situation.

## 5. Account metadata sync without event offline sync

**Problem.** Colours, names and scheduling preferences should follow the user across reinstalls, but copying every recurring series locally would be heavy.

**Approach.** Persist accounts on device, post changes to the backend profile sync API whenever local customisation saves, and optionally refresh account rows when preference screens open. Events stay ephemeral in memory keyed to the current fetch window, reloading after mutations or period changes.

**Trade-offs.** No calendar grid at zero connectivity unless cached data from the last fetch still happens to be in memory; the product targets online scheduling rather than offline diary use.

**Outcome.** Assistant headers always attach fresh account JSON because account state is the durable sync unit, not each event row.

## 6. Token death versus transient errors

**Problem.** Provider refresh tokens revoke when passwords change or sessions are cleared; treating every 401 as a generic error leaves users stuck.

**Approach.** Track accounts whose refresh definitively failed, surface dedicated reconnect sheets in chat and calendar flows, and map backend session-expired responses to the same UX. Successful reconnect reloads accounts before the next turn.

**Trade-offs.** Requires user action; automatic silent recovery is limited to access-token expiry with a valid refresh.

**Outcome.** Chat turns fail closed with actionable reconnect instead of repeated silent errors.

## 7. Local daily brief without server scheduling

**Problem.** Users want a morning summary notification at a chosen clock time, including after reboot, without operating a calendar cron service for every install.

**Approach.** Schedule local notifications from fixed template copy, expose toggles and time picker in settings, and register background work to rebuild schedules when preferences change or the OS restarts workers.

**Trade-offs.** Delivery timing follows OS alarm and battery policies; exact-time guarantees vary by platform settings.

**Outcome.** Predictable on-device scheduling decoupled from assistant availability.

## 8. Plan limits enforced server-side

**Problem.** Client-only flags cannot protect paid assistant usage or action quotas.

**Approach.** Backend returns usage structures in stream events and HTTP 429 bodies; the client shows remaining counts and upgrade sheets. Execute endpoints enforce monthly scheduling-action limits separately from daily chat turns.

**Trade-offs.** Numeric limits are tuning values and change with product decisions; this repository intentionally omits them.

**Outcome.** Free and paid tiers differ in backend behaviour, not just hidden UI flags.

## 9. Automated coverage where layout regressions hurt

**Problem.** Conflict layout and confirmation modals are easy to break with design tweaks.

**Approach.** Maintain focused widget tests on conflict grids, overlap pickers, confirm sheets, greeting copy, preferred-hours clipping and scroll-to-now behaviour, alongside smaller unit tests for copy selection helpers.

**Trade-offs.** Tests do not cover full end-to-end OAuth or live backend streaming; those remain manual or external QA.

**Outcome.** Thirteen test files give fast feedback on the most visual scheduling-critical widgets.
