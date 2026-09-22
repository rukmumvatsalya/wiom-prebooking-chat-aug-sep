# Context & methodology

How the **pre-booking** Aug/Sep report was built and where to be careful. Read this before quoting a figure.

---

## Windows

**10–30 August 2026 (21 days)** vs **1–20 September 2026 (20 days)**, IST. Unequal length, so every
figure is a rate — per 1,000 new users, per day, or per person. Raw totals do not compare.

## Data sources

| Source | What it provides |
|---|---|
| `PROD_DB.DYNAMODB_READ.INTENT_CLASSIFICATIONS` | The chat. Pre-booking rows are those where `APP_PAGE_NAME IS NOT NULL`. 21,286 rows pulled for 10 Aug–20 Sep. |
| `PROD_DB.DBT_CUSTOMER_POD.FCT_CUSTOMER_APP_OPENS` | The denominator — first-ever app open per mobile. |

Pulled via the Metabase API (`metabase.wiom.in`, Snowflake db 113) on 21 September 2026.

## The denominator changed from the June/July report

That report used app-analytics install counts supplied by hand: 48,118 (June) / 49,800 (July). Those
figures were not available for these windows. This report derives its denominator from the warehouse
instead — `MIN(LAUNCH_DATE)` per mobile in `FCT_CUSTOMER_APP_OPENS`:

- **10–30 Aug 2026:** 32,849
- **1–20 Sep 2026:** 62,743

The series is continuous with no gaps and is internally consistent across both windows. But it measures
a different thing from app-store installs — for reference it gives 35,005 for 8–30 June against the
48,118 that report used. **Per-1,000 rates here cannot be compared to the June/July report.**

Note the table starts 2026-05-17, so May first-opens are inflated by the table's own start and should
not be used.

## Timezone, identity, de-duplication

- `CREATED_AT` is UTC → always `DATEADD(minute, 330, …)` for IST.
- `_FIVETRAN_DELETED` is **not** filtered. DynamoDB TTL is 7 days, so filtering silently keeps only the
  last week.
- Pre-booking `MOBILE` is a 6-digit booking-service id, **not a phone number**. It does not join to the
  post-booking identity space or to bookings. This is why the report cannot say whether chatters installed.
- No duplicate rows were found in these windows (`LOG_ID` and millisecond timestamps are unique).

## Naming: `j2_` is a screen set, not the booking variant

The new pages are recorded as `j2_edu1`, `j2_cost` and so on. That prefix is **unrelated to the booking
variant `J2`** in `BOOKING.GROUP_NAME`, which was effectively retired over these windows (302 bookings in
the August window, 6 in September) while the `j2_*` pages carried 6,869 messages in September. Customers on
these screens sit across every live booking variant — I3, J3, J4N, J4R. This report compares *screen sets*;
the booking-variant analysis lives in the install report.

## The rollout confound — the most important caveat

The `j2` pages ramped from ~17 August, crossed over the old flow around 19–23 August, and dominate by
September. Both flows were live every day of both windows.

**August is mostly the original pages; September is mostly the new screens.** An Aug→Sep movement therefore contains a
product change as well as a calendar change, and the two cannot be fully separated. The "Old flow vs j2"
section compares the flows *inside September alone*, which removes the calendar — that is the cleaner read.

Page roster changes to know about:

| Page | Note |
|---|---|
| `HOW_DOES_IT_WORK` | Effectively dead after 5 Sep |
| `j2_landing` | Only 10–22 Aug |
| `j2_user_details` | Only 10–15 Aug |
| `j2_plans` | Started 5 Sep |
| `booking_recharge_options_page` | Started 8 Sep |

Do not read trends into any of these.

## Metric definitions

- **Chip** — identical message text sent by 3+ distinct people, length ≥12 characters. This deliberately
  catches chips the LLM also classified as free text; defining a chip as "`CLASSIFIED_INTENT IS NULL`"
  misses them (it undercounts by ~3pp, and misses `स्पीड कितनी मिलती है?` entirely, which is the single
  most-sent message in the data).
- **Dead end** — a message resolved to `FallbackIntent`.
- **Frustrated** — share of messages the production model tagged `FRUSTRATED` or `ANGRY`.
- **Got an answer** — share of chatters who never hit a dead end in the window.
- **Chip → free text** — share of chip taps immediately followed by a typed message from the same person.
  Not a clean failure rate: some follow-ups are a new question, not a restatement.

## Limitations

- **No page-view data.** Message counts reflect traffic as much as clarity; there is no "share of people
  on this page who asked something".
- **Silent chat opens are invisible.** Only messages are recorded.
- **Thin pages.** `j2_plans` (143 msgs), `j2_slot_selection` (117), `booking_recharge_options_page` (11),
  `NET_COST_EXPLANATION` (5), `HOW_DOES_IT_WORK` (8) are marked thin in the report. The frustration gap on
  the two j2 pages is large and consistent across both, but treat exact percentages as indicative.
- **Sentiment is a model label,** and only the customer side of each conversation is stored.

## Privacy

Source data contains real customer text. Every verbatim is limited to phrases used by 2+ distinct people
and scrubbed of phone numbers and long digit strings. Raw extracts, SQL and the Metabase key are
git-ignored and never committed.
