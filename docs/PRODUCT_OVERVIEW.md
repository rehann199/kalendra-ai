# Product Overview

Kalendra (listed on the stores as Kalendra AI) is a mobile calendar client with a conversational assistant layered on top. The product goal is to let someone see everything they connected in one place, adjust it manually when they prefer taps and forms, and use plain language when they want help finding time or changing an event. The assistant proposes; the calendar UI and confirm sheets remain the gate for writes.

This document describes what the shipped app contains. It is not store marketing copy and it does not claim scheduling outcomes.

## Primary surfaces

### Assistant home

The default home after sign-in is a chat-style screen dated to the current day. A greeting line and three quick-action chips offer starting points such as summarising the day, finding free slots or rescheduling. The composer supports typed messages, a voice capture control and send. Usage messaging can appear when plan limits apply. Cards in the transcript can show structured results (for example day summaries, slot lists or meeting-created summaries) without replacing the calendar grids.

Voice turns record audio on the device, send it through the same backend streaming path as text, optionally play streamed reply audio as chunks arrive, and remove the recording afterward.

### Unified calendar

A dedicated calendar area provides four tabs: day, week, month and schedule. Navigation moves by day, week or month depending on the tab. The day header can show task counts and a routine entry point. Events render as pills coloured by connected calendar. Tasks appear on the schedule tab with checklist interaction.

Creating or opening an event uses bottom sheets: detail view with RSVP for invitations, guest management for organisers, edit flows, and create sheets for new events. Tasks have parallel create, detail and edit sheets.

### Conflicts and overlaps

When two or more events on the same day overlap in time, the layout groups them. Partial overlaps show orange emphasis, shared tint bands between cards in day view, and badges such as "Partial Overlap" or "N+" that open a picker listing every event in the cluster. When start and end times match exactly, the UI switches to red "Conflict" styling, including dashed dividers between side-by-side cards when the overlap spans calendars. Event detail sheets show a banner when siblings exist, coloured for partial versus full conflict.

An artwork asset in this repository illustrates the full-conflict banner between two stacked cards; live captures in the gallery show month chips and an empty day timeline rather than a populated conflict cluster.

### Profile, calendars and preferences

Users connect calendars through an OAuth flow launched in the system browser and completed via a deep link back into the app. Connected calendars can be renamed, coloured and assigned time zones. Scheduling preference flows capture preferred working hours and off-hours per account, which later trim availability suggestions.

A notifications screen toggles all notifications and the daily brief independently, and lets the user pick a brief time (the screenshot in this repo shows 8:00 AM selected).

Billing and usage screens exist for subscription management; this portfolio does not quote prices or plan names from paywalls.

### Onboarding and sign-in

Sign-in supports email and password, Google and Apple. A policy acknowledgement checkbox appears on the welcome screen. Onboarding collects profile-oriented choices such as role selection before calendar connection steps. Email accounts can require verification before assistant features proceed.

## Platforms

The same Flutter codebase ships on Android and iOS. Store listings are linked from the README. Native projects host OAuth URL schemes, notification plugins, secure storage, billing and Firebase SDKs.

## What Kalendra is not

- Not a replacement for the official Google or Outlook apps for every advanced calendar feature; it focuses on unified viewing, assistant-guided changes and the conflict-aware layout described here.
- Not an unattended agent: calendar mutations from the assistant require confirmation through existing editors or sheets.
- Not a guarantee of conflict-free scheduling; the app surfaces overlaps so the person can decide what to change.

## Screenshots in this repository

All WebP captures are real app screens except the labelled overlap artwork. They were cropped to remove the system status bar, captured with sample or empty calendar data where shown, and re-encoded without metadata. Captions in the README describe only what is visible.
