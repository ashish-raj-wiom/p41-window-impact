# P41 — calibrating the CSP acceptance window

What the parameter should be set to, and why.

**📊 [Read the analysis →](https://ashish-raj-wiom.github.io/p41-window-impact/)**

## What P41 is for

When a booking is allocated to a CSP, P41 is the window he has to respond — either take the job or decline it.
If he does neither, P41 fires, the booking is released, and the system goes looking for another CSP.

The parameter sits on one trade-off:

- **How much time is enough for the CSP to act?** Too short and we pull away bookings the CSP was about to take,
  and reroute work that never needed rerouting.
- **How long can the customer be kept waiting?** Too long and the booking sits with someone who was never going
  to act, while the customer waits and the search for a replacement is delayed.

## Recommendation

### Keep P41 at 6 working hours as the default

The binding constraint is not customer impatience by the clock — it is the **promised slot day**. 69% of customer
cancellations happen only after that day has passed, peaking the day after. So the allocation chain has to finish
before the slot day ends.

The median booking has **12.8 working hours** of runway to that deadline and passes through **2 CSPs**. That
implies **12.8 ÷ 2 ≈ 6.4 working hours** per CSP — the median booking can be offered, time out, be re-offered, and
still be installed on the day it was promised. Six hours sits just inside that.

It also comfortably covers CSP behaviour: **P95 response is 4h 52m in the worst quartile** and 3h 24m in the best.

### But cut it to ~5 hours for same-day slots

The runway is not the same for every booking:

| Promised slot | Share | Runway | CSP attempts that fit at 6h | P41 needed for 2 attempts |
|---|---:|---:|---:|---:|
| **Same day** | 23.8% | 9.9 working h | **1.7** — second attempt overruns | **~5 h** |
| Next day | 65.4% | 16.5 working h | 2.8 | ~8 h |
| 2 days ahead | 6.6% | 26.8 working h | 4.5 | ~13 h |
| 3+ days ahead | 3.7% | 40+ working h | 6.7 | ~20 h |

"Attempts that fit" is runway ÷ 6 — how many complete 6-hour windows the booking has room for before the
slot day ends. 1.7 means one CSP gets a full window and the next only gets 70% of one.

For the 65% promised a next-day slot, 6h is comfortable. For the 24% promised a same-day slot it is too long —
one timeout consumes the runway and any reroute lands after the promised day.

**The strongest version: make P41 a function of the runway the booking actually has** — slot date minus now,
divided by expected chain length — rather than a constant.

### And shorten it for CSPs who never respond

The bottom quartile responds to just **5.2%** of bookings even given six hours, and 3.1% of that arrives within
two. Cutting **their** window to 2h would cost ~**11 tasks** a week while releasing ~**520 bookings four working
hours sooner**, buying back runway for the reroute.

## Why

### Method note: current regime only

All figures come from the period **after** P41 moved to 6 hours. Deliberately not a before/after comparison:

1. **Several changes shipped at once.** The customer-scheduling flow went live ~21 July, one day before the P41
   change. Any pre/post difference mixes the two.
2. **Changing the timer changes the behaviour being measured.** Under a 2-hour window you cannot observe a
   five-hour response — the task was already dead. The old regime's numbers are truncated by its own deadline.
   Calibration needs the distribution of what CSPs *actually do* when given room.

### 1. How fast CSPs respond

Response is heavily front-loaded — 41% inside the first quarter-hour — then a long, slowly decaying tail still
climbing when the 6-hour deadline cuts it off at 62.6%.

Between hour 2 and hour 6 the window buys **+4.61pp** of jobs taken and **+3.89pp** of declines — so **46% of
what the later hours surface is a refusal**. A decline is still useful (it releases the booking with a reason,
faster than a timeout) but it is not progress.

### 2. The variation is in *whether* CSPs respond, not how fast

Splitting the **responses themselves** into quartiles — every response sorted by how long it took, cut into
four equal groups of ~496 bookings:

| Response quartile | Responses | Took | Share of all bookings |
|---|---:|---|---:|
| Q1 — fastest quarter | 496 | under **28 sec** | 15.7% |
| Q2 | 496 | 28 sec – 69 sec | 15.7% |
| Q3 | 495 | 69 sec – 55 min | 15.6% |
| **Q4 — slowest quarter** | 495 | **55 min – 6h** (P90 2h 43m, P95 4h 6m) | 15.6% |
| No response at all | 1,183 | — | 37.4% |

**Three quarters of all responses arrive inside 55 minutes.** The entire case for a multi-hour window rests on the
slowest quarter — and even there P95 is **4h 6m**.

Positive action alone (n=1,323) is tighter: three quarters inside **17 minutes**, slowest quarter 17 min – 5h 57m,
P95 **3h 36m**.

### 3. What each extra hour buys

| P41 set to | Q1 | Q2 | Q3 | Q4 | Overall response | Overall jobs taken |
|---:|---:|---:|---:|---:|---:|---:|
| 1 h | 84.3% | 61.5% | 36.3% | 2.8% | 47.4% | 34.0% |
| 2 h | 92.0% | 75.4% | 42.1% | 3.1% | 54.1% | 37.2% |
| 3 h | 94.4% | 80.0% | 46.8% | 3.9% | 57.1% | 38.9% |
| 4 h | 96.3% | 84.4% | 49.7% | 4.4% | 59.2% | 39.9% |
| 5 h | 98.8% | 86.6% | 52.5% | 4.8% | 61.0% | 41.0% |
| **6 h — current** | **100%** | **89.9%** | **54.6%** | **5.2%** | **62.6%** | **41.8%** |

4h → 6h adds **+3.4pp of response** and **+1.9pp of jobs taken**, worth ~1.8pp of booking-level supply efficiency.

### 4. The real deadline is the promised slot, not the clock

Cancellations in the first few hours are not impatience — they are bookings the customer never intended to keep.
Hazard is **0.55%/hour in the first six hours**, then collapses to **0.08%/hour** between hours 6 and 12. Those
would have cancelled whatever P41 was set to.

| Age of booking | 0–6h | 6–12h | 12–24h | 1–1.5d | 1.5–2d | **2–2.5d** | 2.5–3d | 3–4d | 5–7d | 7–10d |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Cancelled per hour | 0.55% | 0.08% | 0.13% | 0.10% | 0.13% | **0.14%** | 0.13% | 0.10% | 0.08% | 0.02% |

After the mistake-booking spike the hazard troughs, then **rises again across days 1–3, peaking at 48–60 hours**.
Re-cut against the customer's **promised slot date** it becomes obvious what they are waiting for:

| Customer cancelled… | Connections | Share |
|---|---:|---:|
| 1 day before the slot | 59 | 11% |
| On the slot day | 100 | 19% |
| **1 day after the slot passed** | **117** | **23%** |
| 2 days after | 64 | 12% |
| 3 days after | 45 | 9% |
| 4+ days after | 133 | 26% |

**69% of cancellations happen after the promised slot has passed**, biggest single day immediately after. The
customer is waiting for the day he was promised. **That is the guardrail.**

### 5. Calibrating against that guardrail

| Input | Value | How measured |
|---|---:|---|
| **Runway** — booking reaching a CSP → end of promised slot day | **12.8 working h** | median; 24.8 calendar hours |
| **Chain** — distinct CSPs a connection passes through | **2** | median; mean 1.91 |
| **Implied P41** = runway ÷ chain | **6.4 working h** | median booking survives one reroute and still makes its slot |

Two CSPs at 6h consume 12 working hours against a 12.8-hour median runway. That is the case for 6 hours, and it
is tighter than a marginal cost-benefit ratio.

### 6. Where 6 hours is too generous

See the runway table in the recommendation above — same-day slots (24% of bookings) only fit 1.7 P41 cycles.

### 7. Time of day makes no difference

P41 runs on a working-hours clock (9 AM – 9 PM, pausing overnight), so a booking arriving at 8 PM gets the same
effective budget as one arriving at 10 AM. The data confirms it:

| Booking reached CSP at | Bookings | Working hrs left that day | Response rate | Jobs taken | P90 response |
|---|---:|---:|---:|---:|---:|
| 09–12 morning | 602 | 10h 24m | 60.5% | 42.0% | 2h 22m |
| 12–15 midday | 679 | 7h 30m | 63.0% | 40.8% | 2h 17m |
| 15–18 afternoon | 790 | 4h 42m | 62.9% | 39.8% | 2h 10m |
| 18–21 late day | 647 | 1h 24m | 61.4% | 41.1% | 2h 21m |
| 21–24 after close | 267 | 0 | 67.8% | 49.8% | 2h 4m |
| 00–09 before open | 180 | 12h | 63.9% | 44.4% | 1h 43m |

Response rate sits in a 60–68% band regardless of arrival time. **No time-of-day-specific parameter is needed.**

## Definitions

- **Working hours** — 9 AM – 9 PM IST, every day. This is P41's native unit: measured on that clock the window
  is exactly 6.00 hours, with minimum, median and maximum identical.
- **Response** — first of a positive action or a decline; the two are mutually exclusive and sum to the response rate.
- **Positive action** — technician assignment. In the customer-proposed-slot flow the slot is already confirmed
  when the booking reaches the CSP, so assigning a technician is his first required step.
- **Population** — install candidates on customer-proposed-slot bookings created 26 Jul – 1 Aug 2026 (n=3,165
  across 1,654 connections). Quartiles restricted to CSPs with ≥5 bookings that week (237 CSPs, 2,142 bookings).
- **Cancellation** — distinct mobiles with a customer-initiated `cancelled` event, from booking confirmation.
  Bookings confirmed 21 Jul – 2 Aug so each has ≥24h observation.

Data as of 4 August 2026. All times IST. Sources: `csp-tas-service`, `csp-demand-allocation-service`,
`booking_logs` via Snowflake (DB 113).
