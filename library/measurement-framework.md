# Measurement Framework

Lifecycle programs are easy to credit and hard to measure. A cart abandonment email "drives" every purchase that follows it in Braze's conversion reporting, including the ones that would have happened anyway. This framework keeps every program in the kit honest: each one proves incremental value against a holdout, reports the same way, and gets turned off when it stops earning its sends.

## KPI hierarchy

Every program defines three tiers in its `PROGRAM.md`.

| Tier | What it is | Rules |
|---|---|---|
| **Primary KPI** | The single behavior the program exists to change, over a fixed window, measured as lift over control | Exactly one per program (two only when a program has two distinct jobs, like post-purchase: repeat purchase and reviews). Decided before the build. Never changed mid-test. |
| **Secondary metrics** | Diagnostics that explain the primary result | Revenue per entrant, conversion by step, redemption rate, click-through. Used to decide what to test next, not to declare success. |
| **Guardrail metrics** | Harms the program must not cause | Unsubscribe rate under 0.3% per send, complaint rate under 0.1% per send, plus program-specific guardrails (SMS opt-outs, discount share, return rate). A guardrail breach overrides a primary-KPI win. |

Opens are never a primary KPI. Privacy features in major mail clients generate machine opens that make open rates unreliable. Use clicks and downstream behavior.

## Control groups

**Default: 10% global holdout at canvas entry.** Control users enter the canvas, count toward conversion tracking, and receive nothing from it. They still receive every other program and all transactional mail, so the comparison isolates this one program.

- **Size:** 10% balances measurement power against revenue left on the table. For programs with small daily entry volume, a 10% control may take many cycles to read; keep it and wait rather than shrink it, or pool cycles for the readout.
- **Randomization:** use Braze's canvas control group, which assigns at entry. Do not build a holdout from a segment filter such as "user ID ends in 7": it collides with other tests using the same trick.
- **Persistence:** keep the control on permanently for always-on programs. A program that was incremental last year can stop being incremental after a site redesign or a competitor's move. A small permanent holdout is the only way to notice.
- **Global holdout (optional, advanced):** a separate 5% of all users held out of every lifecycle program measures the total value of lifecycle marketing. Worth it once you run five or more programs.

**Transactional exception.** Order confirmations, shipping notices, password resets, receipts, and consent requests (program 02's confirmation email) have no control group. Withholding them breaks the customer experience or makes the user unreachable. When a program mixes both, apply the control to the promotional part: program 02 holds out the reminder, not the confirmation.

## Runtime

- **Minimum runtime: two full cycles** before any readout, where a cycle is the program's attribution window. A 7-day cart program needs at least 14 days of entries plus 7 days for the last entries to mature. A 90-day post-purchase KPI needs cohorts that have completed 90 days.
- Read results only on **matured cohorts**: users whose full attribution window has closed.
- Do not stop a test early because it looks good (or bad) at day 4. Early reads are noisy and biased toward the extremes.
- Seasonal businesses: a cycle that spans a peak (holiday, a big sale) is not comparable to one that does not. Note it in the readout, and do not compare it with an off-peak cycle.

## Attribution windows by program

| Program | Primary KPI window | Conversion deadline in Braze |
|---|---|---|
| 01 Welcome / Onboarding | 30 days from entry | 30 days |
| 02 Double Opt-In | 7 days from entry | 7 days |
| 03 Browse Abandonment | 14 days from entry | 14 days |
| 04 Cart Abandonment | 7 days from entry | 7 days |
| 05 Post-Purchase | 90 days (repeat purchase), 30 days from ask (reviews) | 90 days / 30 days |
| 06 Winback | 30 days from entry | 30 days |
| 07 Churn Prevention | 7 days (save), 90 days (re-churn) | 7 days |
| 08 Sunset Cleanup | 30 days after each cycle (deliverability) | Not applicable |
| 09 Back-in-Stock + Price Drop | 7 days from alert | 7 days |
| 10 Loyalty / Referral | 60 days from ask (referral); until expiry date (points) | 60 days |
| 11 Anniversary / Milestone | 30 days (gift window) | 30 days |
| 12 Re-permissioning | 90 days | Not applicable (unsubscribe delta) |

Braze's conversion events count a conversion for any user who converts within the deadline, whether or not they opened the message. That is correct for lift measurement (treatment vs control, both counted the same way) and misleading as "revenue driven." Report the lift.

## Calculating lift

For each program, each readout:

- **Treatment rate** = converters in treatment / entrants in treatment
- **Control rate** = converters in control / entrants in control
- **Absolute lift** = treatment rate minus control rate (in percentage points)
- **Relative lift** = absolute lift / control rate
- **Incremental conversions** = absolute lift x treatment entrants
- **Incremental revenue** = (revenue per treatment entrant minus revenue per control entrant) x treatment entrants, net of discount cost
- **Confidence:** a two-proportion test on the primary KPI. Report the interval, not only the point estimate. "Lift of 1.8 points, 95% interval 0.6 to 3.0" is a result; "lift of 1.8 points" is a guess.

## Monthly reporting template

One page per program, same layout every month.

```
Program:            04 Cart Abandonment · CANVAS | Cart Abandonment | v1
Owner:              <name>
Period:             <matured cohorts entered between YYYY-MM-DD and YYYY-MM-DD>
Status:             On track / Watch / Action needed

PRIMARY KPI · 7-day cart recovery rate
  Treatment         12.4%   (n = 18,210)
  Control            9.1%   (n =  2,023)
  Absolute lift     +3.3 pts   95% CI [+1.9, +4.7]
  Incremental orders          ~600
  Incremental revenue (net)   $XX,XXX

SECONDARY
  Recovery by step            S1 58% · S2 24% · S3 18%
  Code redemption (S3A)       x.x%
  Discount cost per recovered order   $x.xx

GUARDRAILS (worst single send in period)
  Unsubscribe       0.12%   (limit 0.3%)   OK
  Complaint         0.04%   (limit 0.1%)   OK
  SMS opt-out       0.6%    (limit 1%)     OK

CHANGES THIS PERIOD
  <what changed in the build, copy, or audience, with dates>

NEXT TEST
  <one hypothesis, one variable, expected readout date>
```

The numbers in the template are placeholders showing the format, not benchmarks.

## Testing rules

- **One variable per test.** Change the subject line or the send time or the incentive, not all three. A multi-change test tells you something moved, not what.
- Test inside the canvas with Experiment Paths, keeping the 10% global control outside the experiment so program-level lift stays measurable.
- Write the hypothesis, primary metric, and minimum detectable effect before the test starts. Size the test so it can detect that effect in two cycles; if it cannot, test something bigger.
- Declare winners on the primary KPI, not on clicks.

## Anti-patterns

- **No year-over-year comparisons** as evidence of program impact. Traffic, pricing, assortment, and the economy all changed since last year. Only a concurrent holdout isolates the program.
- **No unmeasured always-on programs.** Every always-on canvas has a control group and a monthly readout. A program nobody measures keeps sending after it stops working.
- **No "revenue influenced" as a headline.** Last-touch or any-touch attribution credits the program with purchases it did not cause. Report incremental revenue.
- **No moving the goalposts.** The primary KPI chosen before launch is the one reported after launch.
- **No declaring a win on opens or clicks.** Those are diagnostics.
- **No stacking tests on overlapping audiences** without accounting for it. Two programs testing incentives on the same users in the same week contaminate each other.
- **No removing the control to "capture more revenue."** If the program is incremental, the 10% holdout costs little. If it is not, you need the holdout to find out.
