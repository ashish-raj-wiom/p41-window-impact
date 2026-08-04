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

### Does that extra response become installs?

Yes — but roughly a third of the headline, because a timed-out booking was never a lost booking.

**Late assignments convert as well as early ones — at every hour.** Task-level install rate, i.e. of tasks that
got a technician assigned, the share reaching `CONNECTION_ACTIVE`:

| Assignment latency | Assigned | Installed | Install rate | 95% CI |
|---|---:|---:|---:|---:|
| <15 min | 987 | 480 | 48.63% | 45.5 – 51.7 |
| 15–60 min | 89 | 44 | 49.44% | 39.3 – 59.6 |
| 1–2 h | 101 | 45 | 44.55% | 35.2 – 54.3 |
| **2–3 h** | 54 | 30 | 55.56% | 42.4 – 68.0 |
| **3–4 h** | 32 | 14 | 43.75% | 28.2 – 60.7 |
| **4–5 h** | 33 | 16 | 48.48% | 32.5 – 64.8 |
| **5–6 h** | 27 | 14 | 51.85% | 34.0 – 69.3 |
| **Pooled** | **1,323** | **643** | **48.60%** | — |

**No deviation.** Chi-square across all seven bands: **χ² = 2.15, df = 6, p = 0.906** — nowhere near significance.
The two-group comparison, <2h (48.34%) vs 2–6h (50.68%), gives p = 0.593. The apparent 43.75–55.56% spread is
purely sample size: the hourly bands hold 27–54 tasks, so their intervals are 25–35pp wide and every one of them
straddles the pooled rate. Bolded rows are the P41 gain — they install like everything else.

**But the counterfactual is not zero.** When a candidate dies on `TIMEOUT_P41`, 78.1% of those connections pick
up a fresh candidate and **15.9% install anyway with somebody else**. So the 146 late assignments would have
produced ~23 installs regardless.

| | |
|---|---|
| Raw uplift — 74 installs on 3,165 candidates | +2.34pp |
| Less what reroute would have recovered (146 × 15.9%) | −0.74pp |
| **Net install-rate gain** | **≈ +1.6pp** |

**Treat +1.6pp as a ceiling.** Reroute recovery rises the *younger* the cohort — 7.9% (mid-Jun), 9.0% (early Jul),
11.1% (mid-Jul), 15.1% (late Jul) — which is the opposite of a censoring artifact, so the 15.9% subtracted here
will likely grow and shrink the net gain further.

Still unmeasured: reroute *latency* — how long the second attempt takes.

### But the customer does not appear to walk away

The obvious place that longer wait would show is **customer-initiated cancellation**. It does not.

Measured at **hourly** resolution, because P41 only changed what happens between hour 2 and hour 6 — day-level
buckets cannot see that. Cohorts split at the **true cutover, 21 Jul 23:17 IST** (last 2.00-working-hour window
issued 23:14:39; first 6.00-hour window 23:20:58), not the 22 Jul day boundary.

| Hours since confirmation | PRE (n=4,389) | POST (n=2,107) |
|---|---:|---:|
| 1 | 2.19% | 1.61% |
| **2 — old P41 closes** | **2.55%** | **2.09%** |
| 4 | 3.01% | 2.66% |
| **6 — new P41 closes** | **3.30%** | **2.94%** |
| 12 | 3.78% | 3.42% |
| 24 | 5.33% | 5.32% |
| **Accrual, hour 2 → 6** | **+0.75pp** | **+0.85pp** |

POST sits below PRE at every hour out to 12 — but those levels are not comparable (different weeks, and
cancellation has been falling all year). Applying the same within-cohort logic used everywhere else: between
hour 2 and hour 6, PRE accrues +0.75pp and POST +0.85pp, so POST loses about **0.10pp more** customers in exactly
the span P41 newly occupies. On 2,107 bookings that is **roughly two customers** — not distinguishable from noise,
and not to be reported as a detected cost.

**By hour 24 the two converge exactly** (5.33% vs 5.32%). POST cancels markedly less early then catches up
completely — the signature of cancellation being *deferred* rather than avoided, which is what you would expect
if the booking is simply held longer before anything visible happens.

Caveats: the customer-proposed-slot flow launched ~21 Jul so it is confounded with the P41 change; and a small
P41 penalty could hide inside the secular decline at this sample size. More POST weeks are the cheapest way to
strengthen or overturn this.

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
