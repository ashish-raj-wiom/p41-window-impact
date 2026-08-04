# P41 2h → 6h — Notif to First CSP Action

Analysis of what changed when the D&A OS acceptance window **P41** was extended from 2 hours to 6 hours
on **22 July 2026**.

**📊 [Read the analysis →](https://ashish-raj-wiom.github.io/p41-window-impact/)**

## Question

Distribution and P50 / P90 / P95 of time from task notification to technician assignment, for install
candidates created 26 July – 1 August 2026, after P41 was set to 6 hours.

## Cohorts

Both anchored on **task creation date**, both **full and unsubset**. Each measures the CSP's *first required
action* under the flow in force that week, because the flow changed between them:

| | PRE 14–20 Jul | POST 26 Jul – 1 Aug |
|---|---|---|
| P41 | 2h | 6h |
| CSP's first action | **slot proposal** | **technician assignment** |
| n created | 4,985 | 3,850 |
| n actioned | 1,649 (33.1%) | 1,502 (39.0%) |

## Headline

**The P41 effect is +4.96pp** — the within-cohort accrual between working-hour 2 and working-hour 6, i.e. the
window the old config used to discard.

| Within (working h) | PRE | POST |
|---|---|---|
| 0.25 | 25.94% | 26.75% |
| 1.0 | 29.89% | 29.64% |
| **2.0 — old P41** | **33.08%** | **32.75%** |
| **6.0 — new P41** | **33.08%** | **37.71%** |
| **Accrual, hour 2 → 6** | **+0.00pp** | **+4.96pp** |

`TIMEOUT_P41` deaths fell 52.5% → 37.4%.

The two weeks are **within ~1pp of each other through hour 2** — POST is marginally *behind* at hours 1 and 2.
They diverge only after the old cutoff, which is what isolates the P41 effect from everything else that
changed. CSPs did not get faster; more of their work survived long enough to count.

### Percentiles

All figures in **working hours** (9 AM – 9 PM IST) — the same clock P41 itself runs on.

| Cohort | P50 | P90 | P95 |
|---|---|---|---|
| PRE — slot proposal (n=1,649) | 0.02 | 0.99 | 1.44 |
| POST — tech assigned (n=1,502) | 0.02 | 3.44 | **5.35** |
| ↳ customer-proposed subset (n=1,323) | 0.01 | 2.25 | 3.60 |

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
