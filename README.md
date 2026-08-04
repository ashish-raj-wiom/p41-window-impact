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

The binding constraint is the **pivot**: the afternoon and evening of the day after the promised slot, where
customer cancellation peaks. The allocation has to finish before it.

Measured to that pivot the median booking has **15.51 working hours** of runway and passes through **1.91 CSP
attempts** — a ceiling of **8.1 working hours** per attempt. But the tighter and more useful test is whether a
*reroute* still fits: **81.6% of bookings can absorb two full 6-hour windows before the pivot, and that collapses
to 47.0% at eight hours.**

**Six hours is the last setting that still leaves room for a second CSP.** It also comfortably covers CSP
behaviour — every quartile's P95 response is inside 5 hours, 3h 24m for the best and 4h 52m for the worst.

### A seventh of bookings cannot be helped by any P41 value

**226 of 1,654 bookings (13.7%) reach their first CSP too late in the day** for even one full window to close
before the pivot. They fit nothing at 2 hours and nothing at 12, which is why the "one attempt fits" column sits
flat at about 85% across the entire range.

That is a scheduling matter, not a timer one. If this segment is worth recovering, the lever is when bookings are
allowed to enter allocation relative to the slot they are promised — not the length of P41.

### And shorten it for CSPs who never respond

The bottom quartile responds to just **5.1%** of bookings even given six hours, and 3.3% of that arrives within
two. Cutting **their** window to 2h would cost ~**10 tasks** a week while releasing ~**526 bookings four working
hours sooner**, buying back runway for the reroute.

## Why

### Method note: current regime only

All figures come from the period **after** P41 moved to 6 hours. Deliberately not a before/after comparison:

1. **Several changes shipped at once.** The customer-scheduling flow went live ~21 July, one day before the P41
   change. Any pre/post difference mixes the two.
2. **Changing the timer changes the behaviour being measured.** Under a 2-hour window you cannot observe a
   five-hour response — the task was already dead. The old regime's numbers are truncated by its own deadline.
   Calibration needs the distribution of what CSPs *actually do* when given room.

### 1. How fast CSPs respond, and what the response is

One stacked-area chart: the two bands are jobs taken and declines, and because they sum exactly to the response
rate at every point, the top of the stack **is** the response curve.

Response is heavily front-loaded — 41% inside the first quarter-hour — then a long, slowly decaying tail still
climbing when the 6-hour deadline cuts it off at 62.6%.

Between hour 2 and hour 6 the window buys **+4.61pp** of jobs taken and **+3.89pp** of declines — so **46% of
what the later hours surface is a refusal**. A decline is still useful (it releases the booking with a reason,
faster than a timeout) but it is not progress.

### 2. The variation is in *whether* CSPs respond, not how fast

Ranking every CSP by **his own response rate** across the week's tasks — 237 CSPs with ≥5 bookings, 2,142 tasks —
and cutting them into four equal groups of CSPs. Percentiles are of the response times those CSPs actually
delivered:

| CSP quartile (by response rate) | CSPs | Tasks | Response rate | P25 | P50 | P90 | P95 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Q1 — best responders (100%) | 60 | 485 | **100%** | 24 sec | 38 sec | 1h 37m | 3h 24m |
| Q2 (80–100%) | 59 | 545 | 89.9% | 31 sec | 5m 18s | 3h 11m | 4h 23m |
| Q3 (20–80%) | 59 | 568 | 54.6% | 34 sec | 5m 31s | 3h 48m | 4h 42m |
| **Q4 — worst responders (0–20%)** | 59 | 544 | **5.1%** | 1m 11s | 33m 26s | 4h 12m | **4h 52m** |

**Response rate collapses 100% → 5.1%, a 20× spread.** That is the real difference between CSPs.

**Response speed barely moves in comparison.** The worst quartile's P95 is 4h 52m against the best quartile's
3h 24m — under 90 minutes of spread at P95, against a 6-hour window. Every quartile's median is minutes, not
hours. **No quartile of CSPs needs more than six working hours; the bad quartile simply does not answer.**

### 3. First half: more time buys more response, steadily

| P41 set to | Q1 | Q2 | Q3 | Q4 | Overall response | Overall jobs taken |
|---:|---:|---:|---:|---:|---:|---:|
| 1 h | 84.5% | 61.5% | 35.9% | 2.9% | 47.4% | 34.0% |
| 2 h — old setting | 91.8% | 75.8% | 41.7% | 3.3% | 54.1% | 37.2% |
| 3 h | 94.2% | 80.4% | 46.7% | 3.9% | 57.1% | 38.9% |
| 4 h | 96.3% | 84.6% | 49.5% | 4.4% | 59.2% | 39.9% |
| 5 h | 98.8% | 86.8% | 52.3% | 4.8% | 61.0% | 41.0% |
| **6 h — current** | **100%** | **89.9%** | **54.6%** | **5.1%** | **62.6%** | **41.8%** |
| **Gain per hour, 2h → 6h** | **+2.1pp** | **+3.5pp** | **+3.2pp** | **+0.5pp** | **+2.1pp** | **+1.2pp** |

**Every extra hour buys more response, at a near-constant rate.** The first hour is the steep one; from hour 2
onward each additional hour adds roughly the same again — 2.1pp/h overall. Nothing in the 1h–6h range flattens
out, so there is no point where the next hour stops paying.

**The shape is the same in every quartile; the size is not.** Q1–Q3 gain 2–3.5pp per hour. Q4 gains **0.5pp per
hour**, a seventh of Q2's rate. Waiting buys real response from CSPs who engage at all and almost nothing from
those who do not — the basis for the third recommendation.

**What the move from 2h to 6h was worth.** Response rose **54.1% → 62.6% (+8.5pp)**, jobs taken **37.2% → 41.8%
(+4.6pp)**. Converting to bookings: +4.6pp across **3,165 tasks** ≈ **146 additional first-pass assignments**;
but **44.2%** of connections whose CSP never responded still get a technician from a later CSP, so only 55.8% are
incremental — **≈81 connections** against the cohort's **1,654**, or **+4.9pp of booking-level supply
efficiency**. That is what a return to 2 hours would give back.

### 4. Second half: more time also costs cancellations — but only past one pivot

Cancellations in the first few hours are not impatience — they are bookings the customer never intended to keep,
and they would have cancelled whatever P41 was set to.

| Days since booking | 0.25d | 0.5d | 1d | 1.5d | **2d** | 3d | 4d | 5d | 7d |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Cumulative % cancelled | 3.3% | 3.8% | 5.3% | 6.5% | **8.0%** | 10.9% | 13.0% | 14.9% | 17.9% |

3.3% cancel inside the first six hours, then the curve almost stops — only +0.5pp across the whole of hours 6–12.
It then **steepens again across days 1–3**, climbing 3.8% → 10.9%.

#### 4a. When it happens, relative to the promised slot

Re-cut against the customer's **promised slot date**, all **981** customer-proposed connections pooled, in
**12-hour buckets** (denominator = the 981 connections):

| Bucket | day −1 PM | slot day AM | slot day PM | +1 AM | **+1 PM** | +2 AM | +2 PM | +3 AM | +3 PM |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| % cancelling | 1.63% | 1.63% | 1.43% | 0.61% | **2.45%** | 0.92% | 1.94% | 0.51% | 1.84% |
| cancellations | 16 | 16 | 14 | 6 | **24** | 9 | 19 | 5 | 18 |

Nothing before the promised day beyond a small booking-day spike. The largest single bucket anywhere is the
**afternoon and evening of the day after the slot — 2.45%**, four times the morning bucket before it. Cancellation
then decays with a persistent **PM-heavy sawtooth**: **80 cancellations in 12:00–24:00 buckets against 43 in
00:00–12:00**. Customers wait out the working day and give up in the evening.

The window is cut at +4 days: **139 of the cohort's 172 cancellations fall inside it**, the remaining 33 arriving
later and tailing off. These are rates per bucket, so they do not sum to the cohort's 17.5% overall rate.

**The pivot is the evening of the day after the slot.** Stated precisely: summed to whole days the slot day and
the day after are *level* pooled (3.06% each) — what separates them is concentration, the day after putting 80% of
its mass in one evening bucket. For next-day slots, the majority, the day after is decisively larger (4.17% vs
2.46%).

#### 4b. And it does not depend on which slot he chose

Same 981 connections, same buckets, split by the slot proposed:

| Bucket | day −1 PM | slot day AM | slot day PM | +1 AM | +1 PM | +2 AM | +2 PM |
|---|---:|---:|---:|---:|---:|---:|---:|
| Next day or later (n=572) | 2.80% | 0.52% | 1.92% | 0.70% | **3.15%** | 1.22% | 1.92% |
| Same day (n=409) | 0% | **3.18%** | 0.73% | 0.49% | 1.47% | 0.49% | 1.96% |

Both groups do the same two things: spike once at **their own booking moment** — slot-day morning for same-day
slots, the evening before for later ones, which is the mistake-booking effect — then cluster again **after the
promised day, in the PM buckets**. The post-slot peak for later slots is +1 PM (3.15%); for same-day slots the
post-slot buckets run 6, 8 and 7 cancellations, too close to separate. **The shape does not change with the slot
chosen — only the position of the booking spike, and that moves because the booking moment moves.**

| Slot the customer proposed | Connections | Share of cohort | Cancelled | Before the slot | On the slot day | Day after |
|---|---:|---:|---:|---:|---:|---:|
| Same day | 409 | 41.7% | 14.9% | 0% | **3.9%** | 2.0% |
| **Next day** | **528** | **53.8%** | **20.1%** | 2.8% | 2.5% | **4.2%** |
| 2 days out | 28 | 2.9% | 14.3% | *4 cancellations — too few to rate* | | |
| 3 days out | 16 | 1.6% | 6.3% | *1 cancellation — too few to rate* | | |
| 4+ days out | 0 | 0% | — | *nobody proposes a slot this far out* | | |

**Every percentage in this section uses the same denominator — connections in that row's group**, never
cancellations. So the table's 3.9% and 4.2% are the same numbers the buckets above plot, summed to whole days.

**95.5% of customers propose today or tomorrow.** Noted for context rather than used — it matters for
calibration as much as the cancellation shape: the runway P41 must fit inside is short for essentially the entire
base, not just a tail. It also caps what this section can say about long lead times — 2- and 3-day slots are 44
connections and 5 cancellations between them, a count rather than a rate.

**That is the whole purpose of this split.** It rules out the objection that the pivot is an artefact of slot
mix — that the peak sits where it does only because most customers choose the next day. It does not: each group
pivots on *its own* promised slot. With that established, **the split has done its job and the calibration does
not need it** — section 5 works on the whole cohort and a single median.

**Two cohorts — read them separately.** This section runs on the **cancellation cohort** (981 connections,
25 Jun – 30 Jul, matured), because cancellation needs the slot several days past before it can be counted.
Section 5 runs on the **response cohort** (1,654 connections, 26 Jul – 1 Aug), the same population as sections
1–3. The slot mix differs a little; what carries across is the pivot, not the shares.

**Population for this section.** Unlike the response analysis (26 Jul – 1 Aug), the cancellation cohort runs
**25 Jun – 30 Jul 2026** — cancellation is a slow signal and needs the slot to be several days past before it can
be counted, so the POST-only window is too recent to supply it. One row per connection: its *first* candidate and
the slot promised at that point, so re-slotting and rerouting do not double-count. Only slots ≥5 days past as of
4 Aug are included. The behaviour measured — customers give up when the promised day passes — is not specific to
the P41 window, which is why the wider window is acceptable here.

### 5. Putting the two halves together

**Section 3 established:** every extra hour buys more response, at a near-constant 2.1pp/h, in every quartile.
Nothing in 1h–6h flattens out. On its own that argues for a longer window.

**Section 4 established:** the offsetting cost is not paid by the hour — it lands in one bucket, the **afternoon
and evening of the day after the promised slot**, and it does so whatever slot the customer chose. **The pivot is
PM of the day after the slot.**

**So the two do not trade off smoothly.** Extra hours are close to free right up until the allocation stops
fitting ahead of that pivot, and expensive after. Set P41 to the largest value that still lets the *median*
allocation conclude before it — not to where marginal gain equals marginal cost, because that point does not
exist.

Taking the pivot as **12:00 on the day after the slot** and measuring the whole cohort — no split by slot type:

| Input | Value | How measured |
|---|---:|---|
| **Runway** — first CSP contact → the PM pivot | **15.51 working h** | median across all 1,654 connections; 12.51 h to the end of the slot day plus the following morning's 3 working hours |
| **Chain** — CSP attempts a connection passes through | **1.91** | mean candidates per connection; each consumes one full P41 window |
| **Implied P41** = runway ÷ chain | **8.1 working h** | the ceiling, if all that is required is that the median booking finishes its chain |

8.1 is a ceiling rather than a target, because it only asks the *median* chain to finish. The tighter question is
whether a booking that times out once still has room to be re-offered and answered:

| P41 | One attempt fits | Two attempts fit | What that means |
|---:|---:|---:|---|
| 2 h | 86.0% | 85.4% | room to spare, but gives up 8.5pp of response |
| 4 h | 85.4% | 84.8% | still comfortable |
| **6 h — current** | **85.1%** | **81.6%** | **the last setting that still absorbs a reroute** |
| 8 h | 84.8% | **47.0%** | a timeout now costs the slot for half the base |
| 10 h | 83.3% | 31.0% | — |
| 12 h | 81.6% | 10.8% | — |

**Six hours is the last setting that still leaves room for a reroute.** Fitting one attempt is easy anywhere in
the range — flat at about 85% — because the bookings that never fit are not failing on the parameter. What binds
is the *second* attempt: **81.6% of bookings absorb two full 6-hour windows before the pivot, collapsing to 47.0%
at eight hours.** Below 6h nothing is gained on the promise side, and section 3 has already priced what is lost
on the response side.

The 14% that fit nothing at any setting reach a CSP too late in the day for a single window to close before the
pivot — **226 of 1,654 (13.7%)**. No value of P41 reaches them, which is why the "one attempt fits" column barely
moves across the whole range.

### 6. Time of day makes no difference

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
  Two cuts on different cohorts. *By age of booking:* distinct mobiles, confirmed 1–20 Jul 2026 observed to
  4 Aug, n=4,193. *Aligned on the promised slot:* one row per connection, customer-proposed only, first candidate
  and the slot promised then, slot ≥5 days past as of 4 Aug — 25 Jun – 30 Jul 2026, n=981.

Data as of 4 August 2026. All times IST. Sources: `csp-tas-service`, `csp-demand-allocation-service`,
`booking_logs` via Snowflake (DB 113).
