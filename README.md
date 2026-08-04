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

### Cut P41 from 6 to 2 working hours

The binding constraint is the **promised slot day**. Cancellation steps up sharply on that day and stays up — the
promise is to install *on* it, so the allocation has to conclude **before it begins**.

Measured to that deadline, the median booking for which the constraint is achievable at all has **3.87 working
hours** of runway and passes through **1.91 CSP attempts**. That implies **3.87 ÷ 1.91 ≈ 2 working hours**.

At the current six hours, **only 26.5% of bookings can complete even one CSP attempt before the promised day
starts**. At two hours that rises to **42.8%**, and to **68.5%** among bookings with any runway at all. Two hours
still captures the great majority of responders — every CSP quartile's median response is under 34 minutes.

**The cost, stated plainly:** −8.5pp of response, −4.6pp of jobs taken, roughly **−4.9pp of booking-level supply
efficiency**. That is the price of holding the promised day.

### No P41 value fixes same-day slots — that needs a scheduling change

**37.5% of customers ask for a slot today.** For them the deadline has already passed by **2.93 working hours**
when the booking first reaches a CSP — there was never a "before the slot day" to allocate in. Across the whole
cohort **47.2% of bookings are past the deadline on arrival**.

| Promised slot | Share | Median runway to the pivot | Attempts at 2h | at 6h |
|---|---:|---:|---:|---:|
| **Same day** | 37.5% | **−2.93 working h** | **0** | **0** |
| Next day | 54.8% | 3.87 working h | 1 | **0** |
| 2 days out | 4.1% | 14.2 working h | 7 | 2 |
| 3+ days out | 3.7% | 28.3 working h | 14 | 4 |

Shortening the timer cannot recover time that does not exist. The lever is the promise, not the parameter: stop
offering same-day slots after a cut-off hour, or treat a same-day booking as best-effort rather than dated.

**The strongest version for everyone else: make P41 a function of the runway the booking actually has** — slot
date minus now, divided by the chain length you want to absorb — rather than a constant. One global number is
currently too long for the 92.3% asking for today or tomorrow and needlessly short for the rest.

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

Re-cut against the customer's **promised slot date**, all **981** customer-proposed connections pooled,
**by day** (denominator = the 981 connections):

| Days from promised slot | −2 | −1 | **slot day** | **+1** | +2 | +3 | +4 | +5 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| % cancelling | 0.10% | 1.63% | **3.06%** | **3.06%** | 2.85% | 2.35% | 1.22% | 1.53% |
| cancellations | 1 | 16 | **30** | **30** | 28 | 23 | 12 | 15 |

Before the promised day, almost nothing — 1 cancellation two days out and 16 the day before, against 981
connections. Then a sharp step up that holds: **the promised day and the day after are the two largest days,
level at 3.06% each**, together carrying a third of every cancellation in the window. From +2 it decays steadily.

**The pivot is the slot date itself** — cancellation is near-zero right up to it and steps up the moment it
lands. Since the promise is to install *on* that day, the allocation must be finished before it begins. (Pooled,
day 0 and +1 are level rather than one clearly higher — a composition effect that 4b resolves.)

#### 4b. And it does not depend on which slot he chose

Same 981 connections, same days, split by the slot proposed:

| Days from promised slot | −2 | −1 | slot day | +1 | +2 | +3 | +4 | +5 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Next day or later (n=572) | 0.18% | 2.80% | 2.45% | **3.85%** | 3.15% | 2.45% | 1.40% | 2.10% |
| Same day (n=409) | 0% | 0% | **3.91%** | 1.96% | 2.45% | 2.20% | 0.98% | 0.73% |

Both groups do the same thing: flat before the promise, a step up at it, then a slow decay. **Later slots peak on
+1, the day after the promise (3.85%).** **Same-day slots peak on day 0 (3.91%)** — but for them day 0 *is* the
booking day, so that point carries the mistake-booking spike too and is not comparable like-for-like; their own +1
is a noisier 1.96% on 8 cancellations. Neither shows anything before the promise falls due. **The trend does not
change with the slot chosen.**

| Slot the customer proposed | Connections | Share of cohort | Cancelled | Before the slot | On the slot day | Day after |
|---|---:|---:|---:|---:|---:|---:|
| Same day | 409 | 41.7% | 14.9% | 0% | **3.9%** | 2.0% |
| **Next day** | **528** | **53.8%** | **20.1%** | 2.8% | 2.5% | **4.2%** |
| 2 days out | 28 | 2.9% | 14.3% | *4 cancellations — too few to rate* | | |
| 3 days out | 16 | 1.6% | 6.3% | *1 cancellation — too few to rate* | | |
| 4+ days out | 0 | 0% | — | *nobody proposes a slot this far out* | | |

**Every percentage in this section uses the same denominator — connections in that row's group**, never
cancellations. So the table's 3.9% and 4.2% are the same numbers the buckets above plot, summed to whole days.

**95.5% of customers propose today or tomorrow**, and nobody at all proposes four days out. That matters for
calibration as much as the cancellation shape: the runway P41 must fit inside is short for essentially the entire
base, not just a tail. It also caps what this section can say about long lead times — 2- and 3-day slots are 44
connections and 5 cancellations between them, a count rather than a rate.

**Population for this section.** Unlike the response analysis (26 Jul – 1 Aug), the cancellation cohort runs
**25 Jun – 30 Jul 2026** — cancellation is a slow signal and needs the slot to be several days past before it can
be counted, so the POST-only window is too recent to supply it. One row per connection: its *first* candidate and
the slot promised at that point, so re-slotting and rerouting do not double-count. Only slots ≥5 days past as of
4 Aug are included. The behaviour measured — customers give up when the promised day passes — is not specific to
the P41 window, which is why the wider window is acceptable here.

### 5. Putting the two halves together

**Section 3 established:** every extra hour buys more response, at a near-constant 2.1pp/h, in every quartile.
Nothing in 1h–6h flattens out. On its own that argues for a longer window.

**Section 4 established:** the offsetting cost is not paid by the hour — it steps up at the promised slot day
and stays up. The promise is to install *on* that day, so the allocation must be finished **before the slot day
begins**, not merely before it ends. **The pivot is the slot date itself.**

**So the two do not trade off smoothly.** Extra hours are close to free right up until the allocation stops
fitting ahead of the promised day, and expensive after. Set P41 to the largest value that still lets the *median*
allocation conclude before that pivot — not to where marginal gain equals marginal cost, because that point does
not exist.

Measuring the runway to the **end of the working day before the slot** rather than the end of the slot day:

| Slot the customer proposed | Connections | Share | Median runway to the pivot | Already past the deadline on arrival |
|---|---:|---:|---:|---:|
| **Same day** | 620 | 37.5% | **−2.93 working h** | **620 — all of them** |
| Next day | 906 | 54.8% | +3.87 working h | 160 (17.7%) |
| 2 days out | 67 | 4.1% | +14.2 working h | 0 |
| 3+ days out | 61 | 3.7% | +28.3 working h | 0 |
| **Whole cohort** | **1,654** | 100% | **+0.51 working h** | **780 (47.2%)** |

**Nearly half the base is past the deadline before a CSP ever sees it.** Dividing runway by the chain — **1.91
candidates per connection**, each consuming one full P41 window:

| Population | Median runway | ÷ chain | Implied P41 |
|---|---:|---:|---|
| Whole cohort | 0.51 working h | 1.91 | 0.27 working h — **not a usable parameter** |
| **Achievable subset** (next day or later, 1,034 conns, 62.5%) | **3.87 working h** | 1.91 | **2.03 → 2 working hours** |

Share of bookings whose allocation would conclude before the promised day begins, at each setting:

| P41 | All: 1 attempt | All: 2 attempts | Next day or later: 1 | Next day or later: 2 |
|---:|---:|---:|---:|---:|
| 1 h | 47.0% | 42.8% | 75.1% | 68.5% |
| **2 h** | **42.8%** | **34.7%** | **68.5%** | **55.5%** |
| 3 h | 38.6% | 26.5% | 61.8% | 42.4% |
| 4 h | 34.7% | 16.4% | 55.5% | 26.2% |
| 5 h | 31.0% | 10.0% | 49.6% | 16.1% |
| 6 h — current | 26.5% | 9.3% | 42.4% | 14.9% |

**At six hours only 26.5% of bookings can complete even one attempt before the promised day starts**, and 9.3%
can fit two. Cutting to two hours takes that to 42.8% and 34.7% — and among bookings for which the constraint is
achievable at all, from 42.4% to 68.5%. The cost is section 3's: −8.5pp response, −4.6pp jobs taken, ≈−4.9pp
booking-level supply efficiency.

### 6. The same picture, per slot type

See the runway table in the recommendation above. Under the slot-date pivot a single global number fails at both
ends: too long for the **92.3%** asking for today or tomorrow — neither can fit one 6-hour attempt at the median —
and needlessly short for the **7.8%** asking for two or more days out, who have 14–28 working hours to play with.

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
