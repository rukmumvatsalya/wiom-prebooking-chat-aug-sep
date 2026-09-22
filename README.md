# Pre-booking help chat — August vs September 2026

Two sets of booking screens were live at once, and they get asked different questions.

Analysis of the Wiom in-app support chat across the pre-booking pages,
comparing **10–30 August 2026** against **1–20 September 2026**, counted against first-time app opens.

**[Read the report →](https://rukmumvatsalya.github.io/wiom-prebooking-chat-aug-sep/)**

## Scope

- New users only — pre-booking chat is unreachable for existing customers, so they cannot appear
- 20,752 messages from 7,755 people across both windows
- Denominators: 32,849 first-time app opens (10–30 Aug) and 62,743 (1–20 Sep)

## The headline

A new set of booking screens rolled out from mid-August, crossing over the original pages around
19–23 August. Both ran side by side throughout. The new screens shift the conversation from logistics
("where is my order", "how much") to capability ("what speed", "how many devices", "whole house?").

**Naming:** these pages are recorded with a `j2_` prefix. That is a screen-set name and is **not** the
booking variant `J2`, which was effectively retired over these windows (302 bookings in the August window,
6 in September). For the booking-variant cut see the
[install report](https://rukmumvatsalya.github.io/wiom-postbooking-install-aug-sep/#variants).

Chat per 1,000 new users fell 106.3 → 67.9, but new users nearly doubled — read that as dilution
first, clarity second.

The clearest fixable finding: `j2_plans` and `j2_slot_selection` shipped without suggestion chips and
carry roughly triple the frustration of the pages either side of them.

## Build

Single self-contained HTML file — no scripts, no external requests. Open `index.html` or serve the directory.

## Methodology

See [CONTEXT.md](CONTEXT.md) — sources, metric definitions, the denominator change, the rollout
confound, and the limits.
