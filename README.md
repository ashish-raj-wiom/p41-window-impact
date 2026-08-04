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

### Keep P41 at 6 working hours

It is close to correctly calibrated. Six working hours captures **95%+ of every CSP quartile's responses**, and
those hours are cheap in customer patience: the cancellation hazard is **1.56%/hour in the first hour but
0.09%/hour by hour six**. Cutting to 4h would forfeit ~1.9pp of positive action — roughly **1.8pp of
booking-level supply efficiency** — to save about 0.28pp of cancellation.

**Do not extend beyond 6h on current evidence.** The window is binding, so what CSPs would do at hour 7+ is
unobservable. The marginal return is also decaying: +1.82pp/h of positive action at hour 2–3, +0.85pp/h by 5–6.

### Then: make it depend on the CSP, not one number for everyone

The bottom quartile responds to just **5.15%** of bookings even given the full six hours — and 3.13% of that
arrives within two hours. Cutting **their** window to 2h would cost about **11 tasks** across the week while
releasing roughly **520 bookings four working hours sooner**.

The top quartile already responds to 92% within two hours, so the window rarely binds for them either.
**The 6-hour window earns its keep almost entirely in the middle two quartiles.**

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

| CSP quartile | CSPs | Bookings | Response rate | P25 | P50 | P90 | P95 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Q1 — best | 60 | 485 | 100% | 0.01 | 0.01 | 1.68 | 3.44 |
| Q2 | 59 | 545 | 89.9% | 0.01 | 0.09 | 3.05 | 4.23 |
| Q3 | 59 | 568 | 54.6% | 0.01 | 0.09 | 3.86 | 4.70 |
| **Q4 — worst** | 59 | 544 | **5.2%** | 0.02 | 0.46 | 4.21 | 4.87 |

Working hours, among bookings that got a response. Median response is under 30 minutes in every quartile —
**when a CSP responds at all, he is usually fast.** P95 never exceeds 4.87 working hours, which is what makes 6h
a sufficient ceiling.

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

### 4. What each extra hour costs

Customer cancellation hazard — of bookings still live at the start of each hour, the share cancelled during it:

| Hour of waiting | 0–1 | 1–2 | 2–3 | 3–4 | 4–5 | 5–6 | 6–12 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Cancelled that hour | **1.56%** | 0.46% | 0.23% | 0.33% | 0.19% | **0.09%** | ~0.08% |

Impatience is almost entirely spent in the first hour. The hours P41 is actually deciding about are the cheap ones.

### 5. The trade at the margin

At the hour 5→6 margin, per additional hour: **+0.85pp** jobs taken against **−0.09pp** customers cancelling —
roughly **9 : 1 in favour of waiting**.

Not priced here: **reroute delay.** Every booking that eventually times out now waits six hours instead of two
before the search for a second CSP begins. How much that costs depends on how quickly a released booking finds
someone else — not measured, and the main open question.

### 6. Time of day makes no difference

P41 runs on a working-hours clock (9 AM – 9 PM, pausing overnight), so a booking arriving at 8 PM gets the same
effective budget as one arriving at 10 AM. The data confirms it:

| Booking reached CSP at | Bookings | Working hrs left that day | Response rate | Jobs taken | P90 response |
|---|---:|---:|---:|---:|---:|
| 09–12 morning | 602 | 10.4 | 60.5% | 42.0% | 2.36 |
| 12–15 midday | 679 | 7.5 | 63.0% | 40.8% | 2.28 |
| 15–18 afternoon | 790 | 4.7 | 62.9% | 39.8% | 2.16 |
| 18–21 late day | 647 | 1.4 | 61.4% | 41.1% | 2.35 |
| 21–24 after close | 267 | 0.0 | 67.8% | 49.8% | 2.07 |
| 00–09 before open | 180 | 12.0 | 63.9% | 44.4% | 1.71 |

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
