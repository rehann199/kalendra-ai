# Privacy and Source

## What this repository is

A portfolio showcase for a production application. It describes the engineering of Kalendra; it is not a source distribution, and it is not scheduling or legal advice.

## What Kalendra is and is not

Kalendra is a calendar viewing and scheduling tool with an optional conversational assistant. It reads and writes calendars only for accounts the person connected and only after the flows described in the documentation (manual saves or confirmed assistant actions). It is not a general certification of calendar correctness, and this repository makes no claim that proposed meeting times suit every attendee or timezone edge case.

## What is published

- Written descriptions of architecture, design decisions and trade-offs
- Original diagrams created for this showcase
- Real screenshots from the app, cropped to the app content (system status bar removed), captured with sample or empty data where shown, metadata removed
- One overlap-styling artwork file labelled as artwork in the README
- The app icon, resized to WebP
- Public store links for Android and iOS

## What is intentionally not published

- Production source code, in whole or in part
- Quota numbers, plan limit constants and other tuning values
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration, API keys and tokens
- Ad or attribution account identifiers beyond generic product names
- Billing product identifiers and offer metadata
- Backend hostnames, route names, database paths and security-rule contents
- Language-model prompts and provider credentials
- Version control history of the production repository
- User data of any kind

## Counts and test claims

Line counts cover Dart files under the production app library tree (about 47k lines across 164 files). Test counts refer to thirteen test files; no pass state is claimed. Where behaviour could not be confirmed it is worded conservatively or omitted.

## How the app handles data (conceptual)

This section describes design, not compliance certification, and it does not replace the app's own privacy policy.

- **On the device.** Connected calendar metadata, OAuth tokens, preferences, in-memory assistant transcript and recorded voice bytes (deleted after a voice turn is sent) live in app sandbox storage, with sensitive credential fields using platform secure storage where configured.
- **To the app backend.** Sign-in identity, connected account records (including tokens needed for provider API calls), assistant message text or audio, conversation history bytes resent each turn, device local time zone context, build and subscription context, and mutation payloads for confirmed actions. This is identifiable to the signed-in user, not anonymous aggregate telemetry of calendar contents alone.
- **Through the backend.** Provider calendar and task APIs execute reads and writes the client requests. Assistant turns invoke a language model and speech services; what exactly is forwarded to the model is backend policy and is not documented here.
- **Analytics and crash reporting.** Firebase Analytics, Crashlytics and related SDKs collect usage and stability events; an additional product analytics SDK and an attribution SDK are integrated. This repository does not claim specific data minimisation settings unless verified field-by-field.
- **Push.** Device messaging tokens can be registered with the backend for remote notifications; local notifications schedule daily brief text without requiring the assistant to be open.
- **Subscriptions.** Store purchases flow through native billing APIs; entitlement state is reflected in profile reads used for gating.

## Notes on figures

Diagram boxes summarise behaviour categories. Screenshots show only the screens present in `assets/screenshots/`. No download counts, ratings, revenue or scheduling success rates are stated.

## Images

- **Screenshots** are real captures, re-encoded to WebP without metadata, except the conflict illustration called out as artwork.
- **Diagrams** are original vector illustrations with conceptual labels.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
