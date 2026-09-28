# 06 · Winback

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvas:** `CANVAS | Winback | v1`
> **Owner:** Lifecycle Manager
> **Related programs:** 05 Post-Purchase (upstream), 08 Sunset Cleanup (downstream for anyone winback fails to reactivate), 07 Churn Prevention (subscription businesses use that instead)

## Goal

Reactivate customers whose last purchase was 90 to 180 days ago, and prove the reactivation is incremental against a holdout.

## Why it works

A lapsed customer has not left on purpose. Most drifted: a need was met, a competitor showed up at the right moment, or your emails fell below the fold. Between 90 and 180 days they still remember the brand, still have an open email relationship, and still have a reason to come back if you give them one. After 180 days the odds fall off, and the job shifts from winning them back to cleaning the list (program 08).

The sequence runs from cheapest to most expensive argument. "We miss you" with what is new costs nothing and reactivates the people who only needed a reminder. Bestsellers remove the effort of browsing. The 10% offer comes last and stays flat: an escalation ladder (10%, then 15%, then 20%) teaches lapsed customers that lapsing longer pays better, and they learn fast.

## Entry

| Setting | Value |
|---|---|
| Entry type | Scheduled, daily at 10:00 user local time |
| Trigger | Membership in segment `lapsed-90-180d` at the time of the daily run |
| Trigger properties used | None (scheduled). Personalization uses profile attributes only; never `event_properties`. |
| Entry window | Always on |
| Re-eligibility | 180 days, so a customer who reactivates and lapses again can re-enter, but no one gets two winback streams in the same lapse |

## Audience

**Target:** `lapsed-90-180d` with a reachable, opted-in email address.

**Exclusions:** `unengaged-180d` (program 08 handles them; mailing them hurts deliverability), `subscription-active` (program 07 applies), customers with an open return or support ticket, `internal-test-users`. `discount-abusers` enter but get the no-code variant at step 3.

**Exit criteria:** Makes Purchase (any product); email subscription status becomes `unsubscribed`.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | Day 0 | Email | `EM_WINBACK_S1` | Reminder | "It's been a while." What is new since their last order, personalized by last purchased category. No offer. |
| 2 | Day 7 | Email + Push | `EM_WINBACK_S2`, `PSH_WINBACK_S2` | Remove browsing effort | Top 3 bestsellers in their category, ratings. Push only if push-enabled. |
| 3 | Day 14 | Email | `EM_WINBACK_S3A` / `EM_WINBACK_S3B` | Final argument | A: flat 10% single-use code, 7-day expiry, margin-fenced. B (`discount-abusers`): new arrivals, no code. |

**Per-step notes:** Decision Split on `discount-abusers` before step 3. Quiet hours on. Global frequency caps apply. Incentive: promotion code list `northwind_winback10`, flat 10%, never followed by a deeper offer, excludes sale items and categories under the margin floor. Personalization reads `last_purchased_category` and `last_order_at` from the profile, because scheduled canvases have no trigger event.

## Guardrails

- Flat incentive only. No escalation ladder, ever.
- Do not mail `unengaged-180d`. Winback targets lapsed buyers who still open email; unengaged addresses belong to program 08.
- No guilt or false-urgency copy ("Did we do something wrong?").
- Code expiry shown in the email matches the commerce platform.
- Every Liquid tag has a fallback; category personalization falls back to storewide bestsellers.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Reactivation rate: entrants who purchase within 30 days of entry, treatment vs 10% holdout. Report the lift, not the raw rate; lapsed customers reactivate on their own at a meaningful base rate. |
| Secondary metrics | Revenue per entrant net of discount, share of reactivations before step 3, 90-day second purchase rate of reactivated customers |
| Control design | 10% global control at entry |
| Attribution window | 30 days from entry |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send (watch closely: lapsed audiences complain more) |
| Kill criteria | Complaint rate above 0.1% on any send, or no lift over holdout after two full 30-day cycles |
