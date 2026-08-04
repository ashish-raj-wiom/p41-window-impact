# P41 2h → 6h — Notif to First CSP Action

Analysis of what changed when the D&A OS acceptance window **P41** was extended from 2 hours to 6 hours
on **22 July 2026**.

**📊 [Read the analysis →](https://ashish-raj-wiom.github.io/p41-window-impact/)**

## Question

Distribution and P50 / P90 / P95 of time from task notification to technician assignment, for install
candidates created 26 July – 1 August 2026, after P41 was set to 6 hours.

## Cohorts

Both anchored on **task creation date**. Each measures the CSP's *first required action* under the flow in
force that week, because the flow changed between them. POST is scoped to **customer-proposed-slot bookings**
(where technician assignment genuinely is the CSP's first step); PRE has no equivalent subset because that flow
launched ~21 Jul.

| | PRE 14–20 Jul | POST 26 Jul – 1 Aug |
|---|---|---|
| P41 | 2h | 6h |
| CSP's first action | **slot proposal** | **technician assignment** |
| n created | 4,985 | 3,165 |
| n actioned | 1,649 (33.1%) | 1,323 (41.8%) |

## Headline

**The P41 effect is +4.61pp** — the within-cohort accrual between working-hour 2 and working-hour 6, i.e. the
window the old config used to discard.

| Within (working h) | PRE | POST |
|---|---|---|
| 0.25 | 25.94% | 31.18% |
| 1.0 | 29.89% | 34.00% |
| **2.0 — old P41** | **33.08%** | **37.19%** |
| **6.0 — new P41** | **33.08%** | **41.80%** |
| **Accrual, hour 2 → 6** | **+0.00pp** | **+4.61pp** |

`TIMEOUT_P41` deaths fell 52.5% → 35.5%.

### But only half the gain is progress

A CSP can respond two ways — act, or decline. Both are responses; only one is progress. Splitting the
accrual between working-hour 2 and working-hour 6:

| Accrual, hour 2 → 6 | Response | Positive action | Decline |
|---|---|---|---|
| PRE (P41 = 2h) | +0.02pp | +0.00pp | +0.02pp |
| **POST (P41 = 6h)** | **+8.50pp** | +4.61pp | +3.89pp |
| Share of POST gain | 100% | 54% | **46%** |

**46% of what the longer window surfaces is a refusal, not progress** — and it isn't free. Under the 2-hour
window a CSP who was going to refuse simply timed out at hour 2 and the booking was released for reroute.
Under six hours that same CSP holds the booking and declines at hour 3, 4 or 5, so the customer waits up to
four extra working hours before the task starts looking for anyone else. The same applies to the 35.5% who
still time out: they now hold for six hours instead of two.

Whether the trade is worth it depends on how fast a rerouted booking finds a second CSP — not measured here.

### But the customer does not appear to walk away

The obvious place that longer wait would show is **customer-initiated cancellation**. It does not.

Cohorts anchored on booking confirmation; cancellation measured at fixed horizons from that moment, because
cancellation takes ~80 hours on average and a "has cancelled by now" measure right-truncates recent weeks.

| Cohort | n | ≤6h | ≤24h | ≤72h |
|---|---:|---:|---:|---:|
| PRE — confirmed before 22 Jul | 7,329 | 3.33% | 5.40% | 10.79% |
| **POST — confirmed 22 Jul onward** | 1,337 | **2.77%** | 5.91% | 11.67% |
| Change | | **−0.56pp** | +0.51pp | +0.88pp |

**Inside the six-hour window P41 now holds — exactly where the task used to die at hour 2 and go looking for
someone else — cancellation went down, not up.** The 24h and 72h figures rise slightly, but PRE weeks swing
between 3.45% and 8.04% at 24 hours on their own, so that sits inside week-to-week noise. The 7-day rate has been
falling all year (27% in May → 14% by late July) and the post-change weeks stay on that line.

Three caveats: only **one uncensored POST week** exists (n=1,337); the customer-proposed-slot flow launched ~21 Jul
so it is confounded with the P41 change; and a small P41 penalty could hide inside the secular decline at this
sample size. A second full week is the cheapest way to strengthen or overturn this.

> ⚠️ **`BOOKING_LOGS` duplicates `cancelled` events since 10 July 2026** — 43,985 rows on 13 July across 63
> distinct mobiles, ~700 writes per real cancellation. Counted raw, the series shows a cancellation explosion
> starting twelve days before the P41 change, which reads exactly like a P41 effect. **Always dedupe to distinct
> `MOBILE`.** This will corrupt any other analysis touching the table.

Read the **shape of each curve, never the gap between them**. The cohorts are scoped differently by design, so
POST sits above PRE from the first quarter-hour — before either deadline can bind — and that vertical offset is
cohort composition, not P41. Only the accrual *within* each curve after hour 2 is attributable to the change.
CSPs did not get faster; more of their work survived long enough to count.

### Percentiles

All figures in **working hours** (9 AM – 9 PM IST) — the same clock P41 itself runs on.

| Cohort | P50 | P90 | P95 |
|---|---|---|---|
| PRE — slot proposal (n=1,649) | 0.02 | 0.99 | 1.44 |
| POST — tech assigned (n=1,323) | 0.01 | 2.25 | **3.60** |

PRE's flattering percentiles are **right-truncated by its own 2h deadline** — you cannot observe a 5-hour slot
proposal when the task is killed at hour 2, so POST's higher P95 is a *fuller* measurement, not a worse one.

### P41 is natively a working-hours timer

Measured on a working-hours clock, `P41_DEADLINE_AT − CREATED_AT` is exactly **2.00** before the change and
exactly **6.00** after — minimum, median and maximum all identical to within rounding, zero variance either
side. On a wall clock the same constant budget appears to vary wildly, because the deadline defers past the
9 PM cutoff to the next morning. That is why every figure here is reported in working hours.

In the **customer-proposed flow**, technician assignment *is* the P41-gated action, and there is a hard ceiling
at six working hours — the 6–12, 12–24 and >24h buckets are all empty. The entire long tail (50 assignments)
sits in the other flow, where the CSP proposes a slot first and technician assignment comes after customer
confirmation, outside P41's scope.

## Two filter pitfalls worth knowing

1. **`slot_details:proposedBy` is a flow marker, not a customer attribute.** It takes exactly one value across
   the whole booking table — `CUSTOMER`, on 2,388 rows of 1,104,543; everything else is NULL. It matches 53.1%
   of POST candidates but only 2.1% of PRE, so using it on one side of a comparison silently swaps the
   denominator for "tasks that arrive with a slot already confirmed" and lifts the curve ~7pp at every elapsed
   time. An earlier version of this analysis made exactly that mistake.
2. **`dynamodb_read.booking` is a mutable current-state snapshot.** The identical query 40 minutes apart
   returned 2,083 → 2,045 candidates, 960 → 956 assigned, 713 → 689 P41 deaths. Use
   `SLOT_CONFIRMATION_SOURCE = 'CUSTOMER_PROPOSED'` on the candidate row instead — same intent, immutable, and
   more complete (3,165 vs 2,045).

## Method

- Clock start = `CREATED_AT` (the exact anchor P41 is measured from).
- Clock end = first `TO_STATE='TECHNICIAN_ASSIGNED'` (POST) / first CSP `SLOT_SELECTED` (PRE), reassignments excluded.
- Working hours = monotonic working-time coordinate, `day_index × 43200 + clamp(seconds since 09:00, 0, 43200)`,
  differenced. All seven days counted.
- Timezone verified three ways (`DATEADD +330` on raw TZ, same after `TO_TIMESTAMP_NTZ`, and
  `CONVERT_TIMEZONE`) — all agree exactly.

## Sources

Snowflake (Metabase DB 113):

- `CSP_TAS_SERVICE…INSTALL_EXECUTION_CANDIDATES`
- `CSP_TAS_SERVICE…INSTALL_STATE_TRANSITION_LOG`
- `CSP_DEMAND_ALLOCATION_SERVICE…CONNECTION_ALLOCATIONS`
- `PROD_DB.DYNAMODB_READ.BOOKING` — cross-check only, see pitfall 2

Core table set matches the Health 2.0 Metabase card (#11646). Data as of 4 August 2026. All times IST.
