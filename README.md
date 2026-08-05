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

### Raise P41 from 6 to 8 working hours

The deadline that matters is the **evening after the promised day** — that is when customers give up and cancel.
Working back from it, the average booking has **15.5 working hours** (39.5 calendar hours) before it, and passes
through **1.9 CSPs**. That leaves **8 working hours per CSP**.

Six hours leaves two of those hours unused. CSPs were still accepting jobs at a steady rate when the current
window cut them off, so the extra time should convert — but modestly: roughly **+3 points of response**, worth
about **+0.7 points of system install rate**. For scale, the move from 2 hours to 6 was worth **+2 points**
(section 4).

Those three figures are a projection, not a measurement — see the caveat in section 6. The 8-hour figure itself
is measured.

## Why

Three steps. **Part 1** asks whether a longer window actually buys more response. **Part 2** asks what limits how
long it can be. **Part 3** puts the two together and gets the number. Each part opens with its conclusion, then
the evidence.

### A note on method

**The change did move the numbers.**

| Week | Bookings offered | Responded | Took the job | Responded within 2h |
|---|---:|---:|---:|---:|
| 12–19 Jul — P41 was 2 hours | 5,737 | 46.9% | 33.2% | 46.9% |
| **26 Jul – 1 Aug — P41 is 6 hours** | 3,850 | **59.1%** | **39.0%** | 49.2% |
| **Difference** | — | **+12.2 pts** | **+5.8 pts** | +2.3 pts |

**But that +12.2 cannot be read as the effect of P41**, so the rest of this document uses only the current
period.

- **Several things went live at once**, the customer-scheduling flow among them. A before/after gap cannot be
  attributed to any one of them.
- **Like-for-like the gap is 2.3 points, not 12.2.** At a common 2-hour cut-off the two weeks are 46.9% against
  49.2%. The other ten points are responses arriving in hours 2 to 6 — hours the old window did not have.
- **The two weeks measure different actions** — proposing a slot then, assigning a technician now.

---

## Part 1 of 3 — Does a longer window buy more response?

### Yes — and it was still paying when the current 6-hour window cut it off.

Response climbs steadily: roughly **2 points for every extra hour** from hour 2 onward, and the same pattern
holds inside every group of CSPs. Nothing between 1 and 6 hours flattens out.

What differs between CSPs is not how fast they answer but **whether they answer at all** — 100% in the best
quarter, 5% in the worst. Waiting longer does not close that gap.

**On its own, this argues for a longer window.** Points 1 to 4.

### 1. How fast do CSPs respond?

One stacked-area chart: the two bands are jobs taken and declines, and because they sum exactly to the response
rate at every point, the top of the stack **is** the response curve.

Response is heavily front-loaded — 41% inside the first quarter-hour — then a long, slowly decaying tail still
climbing when the 6-hour deadline cuts it off at 62.6%.

Between hour 2 and hour 6 the window buys **+4.61 points** of jobs taken and **+3.89 points** of declines — so **46% of
what the later hours surface is a refusal**. A decline is still useful (it releases the booking with a reason,
faster than a timeout) but it is not progress.

### 2. Some CSPs answer, some do not

Ranking every CSP by **his own response rate** across the week's tasks — 237 CSPs with ≥5 bookings, 2,142 tasks —
and cutting them into four equal groups of CSPs. Percentiles are of the response times those CSPs actually
delivered:

| CSP group (by how often they answer) | CSPs | Bookings | Responds to | Accepts | Declines | Respond P50 | P90 | P95 | Accept P50 | P90 | P95 | Decline P50 | P90 | P95 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Best quarter | 60 | 485 | **100%** | 68% | 32% | 42 sec | 1h 49m | 3h 26m | 34 sec | 1h 30m | 3h 26m | 4m 35s | 2h 9m | 3h 26m |
| Second | 59 | 545 | 90% | 46% | 44% | 4m 17s | 3h 11m | 4h 25m | 48 sec | 1h 56m | 3h 36m | 30m 40s | 3h 35m | 4h 49m |
| Third | 59 | 568 | 55% | 38% | 17% | 5m 2s | 3h 46m | 4h 40m | 2m 11s | 3h 22m | 4h 9m | 41m 33s | 4h 31m | 5h 10m |
| **Worst quarter** | 59 | 544 | **5%** | 4% | 1% | 33m 26s | 4h 12m | **4h 52m** | 22m 39s | 3h 32m | 4h 52m | 3h 52m | 4h 22m | 4h 25m |

**Response rate collapses 100% → 5.1%, a 20× spread.** That is the real difference between CSPs.

**Response speed barely moves in comparison.** The worst quarter's P95 is 4h 52m against the best quarter's
3h 26m — under 90 minutes of spread at P95, against a 6-hour window. Every quarter's median is minutes, not
hours. **No quarter of CSPs needs more than six working hours; the bad quarter simply does not answer.**

**Accepting is always faster than declining.** In every group the median acceptance lands in under three minutes,
while the median decline runs from five minutes in the best group to nearly four hours in the worst. A CSP who is
going to take the job does it almost immediately; the later hours are mostly spent waiting on people who will
eventually say no, or nothing at all.

#### CSPs on the Dominance programme

The same cut for the 25 CSPs who accepted Dominance in late July, counted only from the day each one joined:

| Group | CSPs | Bookings | Responds to | Takes the job | P50 | P90 | P95 |
|---|---:|---:|---:|---:|---:|---:|---:|
| **Dominance** | 23 | 81 | **86%** | **69%** | 50 sec | 1h 32m | 2h 28m |
| Everyone else | 655 | 3,068 | 62% | 41% | 71 sec | 2h 49m | 4h 7m |

**Dominance CSPs answer far more often — 86% against 62% — and take the job 69% of the time against 41%.** They
are quicker too: P95 of 2h 28m against 4h 7m.

**Two cautions before reading anything causal into this.** The sample is thin — 81 bookings, because the
programme started 27–30 July and the week ends 1 August, giving each CSP two to six days inside the window. And
these CSPs were *selected* for the programme on the strength of their area coverage, so the gap describes who is
in it at least as much as what it has done to them. Read it as a profile of the group, not a measured effect.
(16 bookings that reached a Dominance CSP before he joined are excluded from both rows.)

Resolution: partner ids from the Dominance sheet → `DBT_CSP_POD.INT_EW_CSP_ACCOUNTS` → the `CSP_ID` used on
install candidates. All 25 resolved; 24 were active that week, 23 after their own join date.

### 3. What each extra hour is worth

| P41 set to | Q1 | Q2 | Q3 | Q4 | Overall response | Overall jobs taken |
|---:|---:|---:|---:|---:|---:|---:|
| 1 h | 82.3% | 63.3% | 36.1% | 2.9% | 47.4% | 34.0% |
| 2 h — old setting | 91.1% | 76.1% | 41.9% | 3.3% | 54.1% | 37.2% |
| 3 h | 94.0% | 80.4% | 46.8% | 3.9% | 57.1% | 38.9% |
| 4 h | 96.1% | 84.6% | 49.6% | 4.4% | 59.2% | 39.9% |
| 5 h | 98.8% | 86.6% | 52.5% | 4.8% | 61.0% | 41.0% |
| **6 h — current** | **100%** | **89.9%** | **54.6%** | **5.1%** | **62.6%** | **41.8%** |
| **Gain per hour, 2h → 6h** | **+2.2 points** | **+3.5 points** | **+3.2 points** | **+0.4 points** | **+2.1 points** | **+1.2 points** |

**Every extra hour buys more response, at a near-constant rate.** The first hour is the steep one; from hour 2
onward each additional hour adds roughly the same again — 2.1 points an hour overall. Nothing in the 1h–6h range flattens
out, so there is no point where the next hour stops paying.

**The shape is the same in every quarter; the size is not.** The best three groups gain 2.2–3.5 points per hour. The weakest gains **0.4 points
per hour**, a ninth of the second group's rate. Waiting buys real response from CSPs who engage at all and almost nothing from
those who do not.

**What the move from 2h to 6h was worth.** Response rose **54.1% → 62.6% (+8.5 points)**, jobs taken
**37.2% → 41.8% (+4.6 points)**. Those are task-level figures and they overstate the window's value, because many
of the extra acceptances are second or third attempts on bookings that would have been served anyway. **Section 4
measures it at booking level and gets +2 points of system install rate** — that is the figure to use.

### 4. What the extra hours are worth in installs

Response is not the point in itself — an install is. Per offer made to a CSP:

| P41 | Offers | Accepted | Offers ending in an install | Install rate *of* those accepted |
|---:|---:|---:|---:|---:|
| 2 h | 3,165 | 37.2% | 18.0% | 48.4% |
| 4 h | 3,165 | 39.9% | 19.4% | 48.6% |
| **6 h — current** | 3,165 | **41.8%** | **20.3%** | **48.7%** |

The last column matters most: **an acceptance in hour 5 becomes an install just as often as one in the first
minute — 48.7% either way.** The later hours buy the same quality of work, just later.

Per booking, by how long its first CSP took:

| First response arrived | Bookings | Share | Installed | Install rate |
|---|---:|---:|---:|---:|
| Within 2h | 977 | 59.1% | 495 | **50.7%** |
| 2–4h | 76 | 4.6% | 35 | 46.1% |
| 4–6h | 57 | 3.4% | 27 | 47.4% |
| **Never answered** | 543 | 32.8% | 118 | **21.7%** |

Flat across the response bands, then a cliff. **A booking whose first CSP goes silent still installs 21.7% of the
time** — someone else picks it up — so a timeout costs roughly 25 points of install probability, not all of it.

Only **134 bookings (8.1%)** get their first response between hours 2 and 6 — the entire population the extra
four hours buys:

| P41 set to | Bookings that would time out instead | System install rate |
|---:|---:|---:|
| 2 h | 134 | 38.8% |
| 4 h | 58 | 39.9% |
| **6 h — current** | — | **40.8%** |

**Moving from 2 hours to 6 is worth about 2 points of system install rate.** Real, but well below what the
response numbers alone imply — a third of bookings never get a first answer at any setting, and a fifth of those
still install through someone else.

---

## Part 2 of 3 — So what stops us making it longer?

### A deadline — not a cost that grows by the hour.

Customers do not get impatient hour by hour. They wait for the day they were promised and give up on the
**evening after it**. Cancellation is near zero right up to that point and jumps the moment it arrives.

That fixes the point the allocation must finish before — and it is **the same point whichever day the customer
chose**.

**Extra hours are close to free until that deadline, and expensive after it.** Point 4.

### 5. When customers give up

Cancellations in the first few hours are not impatience — they are bookings the customer never intended to keep,
and they would have cancelled whatever P41 was set to.

| Days since booking | 0.25d | 0.5d | 1d | 1.5d | **2d** | 3d | 4d | 5d | 7d |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Cumulative % cancelled | 2.8% | 3.3% | 5.0% | 6.4% | **9.2%** | 12.2% | 15.3% | 17.1% | 20.7% |

2.8% cancel inside the first six hours, then the curve almost stops — only +0.5 points across the whole of hours 6–12.
It then **steepens again across days 1–3**, climbing 3.3% → 12.2%.

#### 5a. When customers actually cancel

Re-cut against the customer's **promised slot date**, all **981** customer-proposed bookings pooled, in
**12-hour buckets** (base = the 981 bookings):

| Half-day | day −1 aft | slot day morn | slot day aft | +1 morn | **+1 aft** | +2 morn | +2 aft | +3 morn | +3 aft |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| % cancelling | 1.63% | 1.63% | 1.43% | 0.61% | **2.45%** | 0.92% | 1.94% | 0.51% | 1.84% |
| cancellations | 16 | 16 | 14 | 6 | **24** | 9 | 19 | 5 | 18 |

Nothing before the promised day beyond a small booking-day spike. The largest single bucket anywhere is the
**afternoon and evening of the day after the slot — 2.45%**, four times the morning bucket before it. Cancellation
then decays with a persistent **tilt towards afternoons and evenings**: **80 cancellations in 12:00–24:00 buckets against 43 in
00:00–12:00**. Customers wait out the working day and give up in the evening.

The window is cut at +4 days: **139 of the group's 172 cancellations fall inside it**, the remaining 33 arriving
later and tailing off. These are rates per bucket, so they do not sum to the group's 17.5% overall rate.

**The deadline is the evening of the day after the slot.** Stated precisely: summed to whole days the slot day and
the day after are *level* pooled (3.06% each) — what separates them is concentration, the day after putting 80% of
its mass in one evening bucket. For next-day slots, the majority, the day after is decisively larger (4.17% vs
2.46%).

#### 5b. It does not matter which day they chose

Same 981 bookings, same buckets, split by the slot proposed:

| Half-day | day −1 aft | slot day morn | slot day aft | +1 morn | +1 aft | +2 morn | +2 aft |
|---|---:|---:|---:|---:|---:|---:|---:|
| Next day or later (n=572) | 2.80% | 0.52% | 1.92% | 0.70% | **3.15%** | 1.22% | 1.92% |
| Same day (n=409) | 0% | **3.18%** | 0.73% | 0.49% | 1.47% | 0.49% | 1.96% |
| **All bookings (n=981)** | 1.63% | 1.63% | 1.43% | 0.61% | **2.45%** | 0.92% | 1.94% |

Both groups do the same two things: spike once at **their own booking moment** — slot-day morning for same-day
slots, the evening before for later ones, which is the mistake-booking effect — then cluster again **after the
promised day, in the afternoon and evening**. The post-slot peak for later slots is the afternoon after the promised day (3.15%); for same-day slots the
post-slot buckets run 6, 8 and 7 cancellations, too close to separate. **The shape does not change with the slot
chosen — only the position of the booking spike, and that moves because the booking moment moves.**

**That is the whole purpose of this split.** It rules out the objection that the deadline is an artefact of slot
mix — that the peak sits where it does only because most customers choose the next day. It does not: each group
turns on *its own* promised slot. With that established, **the split has done its job and the calibration does
not need it** — section 6 works on the whole group and a single median.

**Two groups — read them separately.** This section runs on the **cancellation group** (981 bookings offered
22–30 Jul, matured), because cancellation needs the slot several days past before it can be counted.
Section 6 runs on the **response group** (1,654 bookings, 26 Jul – 1 Aug), the same population as sections
1–3. The slot mix differs a little; what carries across is the deadline, not the shares.

**Population for this section.** Unlike the response analysis (26 Jul – 1 Aug), the cancellation group runs
**22–30 Jul 2026** — cancellation is a slow signal and needs the slot to be several days past before it can
be counted, so the POST-only window is too recent to supply it. One row per booking: its *first* candidate and
the slot promised at that point, so re-slotting and rerouting do not double-count. Only slots ≥5 days past as of
4 Aug are included. The behaviour measured — customers give up when the promised day passes — is not specific to
the P41 window, which is why the wider window is acceptable here.

---

## Part 3 of 3 — So what should P41 be?

### 8 working hours. We are at 6.

Part 1 says take as much time as the deadline allows. Part 2 says where the deadline is. The setting falls out of
the two: **15.5 working hours available ÷ 1.9 CSPs tried = 8 working hours each.**

### 6. The calculation

**Part 1 established:** every extra hour buys more response, at a near-constant 2.1 points an hour, in every quarter.
Nothing in 1h–6h flattens out. On its own that argues for a longer window.

**Part 2 established:** the offsetting cost is not paid by the hour — it lands in one bucket, the **afternoon
and evening of the day after the promised slot**, and it does so whatever slot the customer chose. **The deadline is
the afternoon and evening of the day after the promised slot.**

**So the two do not trade off smoothly.** Extra hours are close to free right up until the allocation stops
fitting ahead of that deadline, and expensive after. Set P41 to the largest value that still lets the *median*
allocation conclude before it — not to where marginal gain equals marginal cost, because that point does not
exist.

Working back from that deadline across all 1,654 bookings in the week:

| What we measure | Value | How |
|---|---:|---|
| **Time available** — booking reaching its first CSP → the deadline | **15.5 working h**<br>(39.5 calendar h) | the middle booking — about a day and a half of real time, 15.5 hours once the overnight gap is taken out |
| **CSPs tried** — how many get the booking before it is done | **1.9** (median 1) | average per booking; each holds it for a full window. Median 1, P75 2, P90 4, P95 4, longest 9 |
| **Time per CSP** = available ÷ tried | **8 working hours** | the setting at which the typical booking still finishes ahead of the deadline |

**So the answer is 8 working hours, and we are currently at 6.**

**One caveat on the average of 1.9.** The distribution is skewed:

| CSPs tried | Bookings | Share | Running total | Time each, if the 15.5 hours is shared |
|---:|---:|---:|---:|---|
| **1** | 829 | **50.1%** | 50.1% | 15.5 h — the whole runway |
| 2 | 484 | 29.3% | 79.4% | 7.8 h |
| 3 | 128 | 7.7% | 87.1% | 5.2 h |
| 4 | 143 | 8.6% | 95.7% | 3.9 h |
| 5 or more | 70 | 4.3% | 100% | 3.1 h or less — longest chain is 9 |

**Half of all bookings never leave the first CSP.** For them the full 15.5 hours is available and 8 is
comfortable. The average is carried by a tail: the fifth of bookings passing through three CSPs or more already
overrun the deadline at today's 6 hours, and no single setting fixes them — their problem is repeated rerouting,
not the length of each window.

Eight hours is the right system-wide number, but **generous for the half answered first time and irrelevant to
the fifth rerouted three times or more**.

**What the extra two hours should buy — and why this part is an estimate.** We cannot measure it directly: the
timer kills a booking at 6 hours, so no CSP has ever had the chance to answer in hour 7. The response curve looks
flat after 6 hours only because that is where it is cut off.

What we can see is the rate at which acceptances were still arriving when the window closed — **+2.1, +1.8 and
+1.6 points** in the last three hours, slowing but nowhere near stopped. Carrying that same slowdown two hours
further gives **+2.8 points of response**. Run through section 4's booking-level arithmetic — about 46
bookings getting a first answer they otherwise would
not, at a 25-point install differential — that is **+0.7 points of system install rate**, a third of what the
last extension delivered. The direction is not in doubt, only the size, and the size is small.

**Coverage caveat.** 226 of the 1,654 bookings (one in seven) reach a CSP too late in the day for even a single
window to close before the deadline. No timer value reaches them, at 2 hours or at 12.

---

## One last check

### Time of day makes no difference

P41 runs on a working-hours clock (9am–9pm, pausing overnight), so a booking arriving at 8pm gets the same
effective budget as one arriving at 10am. The data confirms it:

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

- **Working hours** — 9am–9pm IST, every day. This is P41's native unit: measured on that clock the window
  is exactly 6.00 hours, with minimum, median and maximum identical.
- **Response** — first of a positive action or a decline; the two are mutually exclusive and sum to the response rate.
- **Positive action** — technician assignment. In the customer-proposed-slot flow the slot is already confirmed
  when the booking reaches the CSP, so assigning a technician is his first required step.
- **Population** — bookings offered to a CSP on customer-proposed-slot bookings created 26 Jul – 1 Aug 2026 (n=3,165
  across 1,654 bookings). Quartiles restricted to CSPs with ≥5 bookings that week (237 CSPs, 2,142 bookings).
- **Cancellation** — distinct mobiles with a customer-initiated `cancelled` event, from booking confirmation.
  Two cuts on different groups. *By age of booking:* customer-proposed bookings offered 22–28 Jul 2026, each observed a full
  7 days, n=642. *Aligned on the promised slot:* one row per booking, customer-proposed only, first candidate
  and the slot promised then, slot ≥5 days past as of 4 Aug — bookings offered 22–30 Jul 2026, n=981.

Data as of 4 August 2026. All times IST. Sources: `csp-tas-service`, `csp-demand-allocation-service`,
`booking_logs` via Snowflake (DB 113).
