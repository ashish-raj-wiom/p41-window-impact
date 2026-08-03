# P41 2h → 6h — Notif to First CSP Action

Analysis of what changed when the D&A OS acceptance window **P41** was extended from 2 hours to 6 hours
on **22 July 2026**.

**📊 [Read the analysis →](https://ashish-raj-wiom.github.io/p41-window-impact/)**

## Question

Distribution and P50 / P90 / P95 of time from task notification to technician assignment, for install
candidates created 26 July – 1 August 2026, after P41 was set to 6 hours.

## Cohorts

Both cohorts are anchored on **task creation date**, and each measures **the CSP's first required action**
in its own regime — because the flow changed between them:

| | PRE 14–20 Jul | POST 26 Jul – 1 Aug |
|---|---|---|
| P41 | 2h | 6h |
| CSP's first action | **slot proposal** | **technician assignment** |
| Cohort | valid Indian mobile | `slot_details:proposedBy = 'CUSTOMER'` + valid mobile |
| n | 4,903 | 2,083 |

The POST cohort is scoped to bookings where the **customer proposed the slot**, so technician assignment
genuinely is the CSP's first step. 99.2% of the selected candidates independently carry
`SLOT_CONFIRMATION_SOURCE='CUSTOMER_PROPOSED'`, which cross-validates the filter.

## Headline

**The P41 effect is +5.5pp more tasks actioned** — measured as within-cohort accrual between working-hour 2
and working-hour 6, the window the old config used to discard.

| | PRE | POST |
|---|---|---|
| Actioned by working-hour 2 | 33.08% | 40.57% |
| Actioned by working-hour 6 | 33.08% | 46.09% |
| **Accrual, hour 2 → 6** | **+0.00pp** | **+5.52pp** |
| Died on `TIMEOUT_P41` | 52.4% | 34.2% |

Each curve flattens exactly at its own P41 deadline. PRE stops dead at hour 2 with tasks still arriving;
POST keeps accruing and only flattens at hour 6, after which **zero** assignments arrive — P41 is a hard
binding ceiling, not a slack constraint.

### Percentiles

| Cohort | P50 cal | P90 cal | P95 cal | P50 work | P90 work | P95 work |
|---|---|---|---|---|---|---|
| PRE — slot proposal (n=1,622) | 0.02 | 1.96 | 8.78 | 0.02 | 0.99 | 1.44 |
| POST — tech assigned (n=960) | 0.03 | 8.33 | 12.22 | 0.01 | 2.44 | **3.61** |

Hours, over every candidate that reached the action. "work" = 9 AM – 9 PM IST clock.

Medians sit near zero because in this flow the slot is already confirmed when the candidate is created —
45% of assignments happen within a minute of notification, since nothing stands between the two.

PRE's flattering percentiles are **right-truncated by its own 2h deadline** — you cannot observe a 5-hour
slot proposal when the task is killed at hour 2. POST's higher P95 is a *fuller* measurement, not a worse one.

## Two things not to conclude

1. **The vertical gap between the curves is not the P41 effect.** POST sits ~7pp above PRE from the first
   quarter-hour, before either deadline can matter — that is cohort composition (customers who actively
   proposed a slot are higher-intent). Applying the one symmetric filter component, valid mobile, moves PRE
   by nothing (33.1% → 33.1%). Only the within-cohort accrual after hour 2 is attributable to P41.
2. **CSPs did not get faster.** The gain is entirely tasks surviving long enough to be actioned.

## Method

- Clock start = `CREATED_AT` (the exact anchor P41 is measured from).
- Clock end = first `TO_STATE='TECHNICIAN_ASSIGNED'` (POST) / first CSP `SLOT_SELECTED` (PRE), reassignments excluded.
- Working hours = monotonic working-time coordinate, `day_index × 43200 + clamp(seconds since 09:00, 0, 43200)`,
  differenced. All seven days counted.
- Timezone verified: `CONVERT_TIMEZONE('Asia/Kolkata', …)` and `+330 minutes` agree on every row.

Full definitions, the exact booking filter, its known snapshot limitation, data lineage and open questions
are in the page's Methodology section.

## Sources

Snowflake (Metabase DB 113):

- `PROD_DB.DYNAMODB_READ.BOOKING` — POST cohort filter
- `PROD_DB.PUBLIC.COMPANY_B_CONNECTION_BOOKING_ENRICHED` — mobile → connection bridge
- `CSP_TAS_SERVICE…INSTALL_EXECUTION_CANDIDATES`
- `CSP_TAS_SERVICE…INSTALL_STATE_TRANSITION_LOG`
- `CSP_DEMAND_ALLOCATION_SERVICE…CONNECTION_ALLOCATIONS`

Core table set matches the Health 2.0 Metabase card (#11646). Data as of 3 August 2026. All times IST.
