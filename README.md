# P41 2h → 6h — Notif to First CSP Action

Analysis of what changed when the D&A OS acceptance window **P41** was extended from 2 hours to 6 hours
on **22 July 2026**.

**📊 [Read the analysis →](https://ashish-raj-wiom.github.io/p41-window-impact/)**

## Question

Distribution and P50 / P90 / P95 of time from task notification to technician assignment, for install
candidates created 26 July – 1 August 2026, after P41 was set to 6 hours.

## Headline

| | PRE (14–20 Jul, P41 = 2h) | POST (26 Jul – 1 Aug, P41 = 6h) |
|---|---|---|
| Tasks actioned | 33.1% | **39.0%** (+5.9pp) |
| Died on `TIMEOUT_P41` | 52.5% | **37.4%** (−15.1pp) |
| Actioned within 2 working hours | 33.1% | **32.8%** (unchanged) |

Extending P41 bought ~6pp more tasks actioned and cut P41 deaths by 15pp — but CSP responsiveness inside
the first two working hours is **identical** across the two weeks. The entire gain is incremental accrual
between hour 2 and hour 6: work that used to be guillotined before the CSP got to it.

P50 / P90 / P95 are reported on both a calendar clock and a **9 AM – 9 PM IST working-hours** clock. On the
working-hours clock the post-change P95 is **6.5 hours** — landing squarely on the new 6h budget.

## Two caveats

1. **43% of post-change assignments are not a CSP acting.** 647 of 1,500 complete within 60 seconds of
   candidate creation, with the whole allocation → candidate → slot-confirmed → technician-assigned chain
   replayed at ingest. Real actors, real downstream outcomes — but not human action. Mechanism unexplained;
   needs an engineering answer.
2. **P41 is confounded with the customer-scheduling launch** (~21 July, one day earlier), now 82% of the
   cohort. Because the customer proposes the slot up front, the CSP's first required action changed from
   *slot proposal* (PRE) to *technician assignment* (POST) — which is why the two cohorts are measured on
   different actions.

## Method

- Cohort anchored on **task creation** (`CREATED_AT`, IST), tracked forward. One row per `EXECUTION_CANDIDATE_ID`.
- Clock start = `CREATED_AT` (the exact anchor P41 is measured from).
- Clock end = first `TO_STATE='TECHNICIAN_ASSIGNED'` (POST) / first CSP `SLOT_SELECTED` (PRE), reassignments excluded.
- Working hours = monotonic working-time coordinate, `day_index × 43200 + clamp(seconds since 09:00, 0, 43200)`,
  differenced. Counted on all seven days.

Full definitions, data lineage and open questions are in the page's Methodology section.

## Sources

Snowflake (Metabase DB 113) — `csp-tas-service` and `csp-demand-allocation-service`. Table set matches the
Health 2.0 Metabase card (#11646).

- `CSP_TAS_SERVICE…INSTALL_EXECUTION_CANDIDATES`
- `CSP_TAS_SERVICE…INSTALL_STATE_TRANSITION_LOG`
- `CSP_DEMAND_ALLOCATION_SERVICE…CONNECTION_ALLOCATIONS`

Data as of 3 August 2026. All times IST.
