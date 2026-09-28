# 10 · Loyalty / Referral

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvases:** `CANVAS | Referral Ask | v1`, `CANVAS | Loyalty Tier Upgrade | v1`, `CANVAS | Loyalty Points Expiry | v1`
> **Owner:** Lifecycle Manager, with the loyalty program owner and fraud team
> **Related programs:** 05 Post-Purchase (the referral ask shares its T+21d slot), 11 Anniversary / Milestone

## Goal

Turn satisfied customers into a measurable source of new customers through referrals, and keep loyalty members active with timely tier and points messages.

## Why it works

A referral ask works when it arrives right after the customer has had a good experience and before the memory fades. Three weeks after purchase, the product has been used, and a customer who left a 4 or 5 star review has just told you they are happy. Two-sided rewards (the friend gets a discount, the referrer gets credit) make the ask feel like a favor to the friend rather than a sales job.

Loyalty messages work on two levers. Tier upgrades are recognition: celebrate them the day they happen, and say exactly what changed. Points expiry is loss aversion, and it only works if the warning arrives early enough to act (30 days) and again when it is urgent (7 days).

## Entry

| Setting | Value |
|---|---|
| Entry type | Referral: action-based on Makes Purchase with a 21-day delay (or a branch inside program 05). Tier: action-based on `loyalty_tier_upgraded`. Expiry: scheduled daily on `points_expiring_30d` and `points_expiring_7d` segments. |
| Trigger properties used | `new_tier`, `old_tier`, `benefits` (tier); profile attributes `points_balance`, `points_expiring`, `points_expiry_date` (expiry, scheduled, so no event properties) |
| Entry window | Always on |
| Re-eligibility | Referral ask: 90 days. Tier: per upgrade. Expiry: per expiry date. |

## Audience

**Target:** Referral: customers with a delivered order and no open return, prioritizing `last_review_rating` of 4 or more. Tier and expiry: loyalty members.

**Exclusions:** Referral: `discount-abusers`, accounts flagged by fraud, customers with an open support ticket. Expiry: members whose expiring points are below a meaningful redemption threshold.

**Exit criteria:** Referral: the customer sends a referral (`referral_sent`). Expiry: points redeemed or expired.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| R1 | T+21d post-purchase | Email | `EM_REFERRAL_S1` | Referral ask | Personal referral link, what the friend gets, what the referrer gets, terms link. |
| R2 | R1 + 7d, if no referral and opened R1 | Push | `PSH_REFERRAL_S2` | Reminder | One line, deep link to share sheet. |
| T1 | On upgrade | Email + In-App | `EM_LOYALTY_TIER_S1`, `IAM_LOYALTY_TIER_S1` | Celebrate | New tier, the specific new benefits, how to use them now. |
| P1 | 30 days before expiry | Email | `EM_LOYALTY_EXPIRY_S1` | Early warning | Points expiring, date, what they can redeem for. |
| P2 | 7 days before expiry | Email + Push | `EM_LOYALTY_EXPIRY_S2`, `PSH_LOYALTY_EXPIRY_S2` | Final warning | Same, plus the exact expiry date and a one-click redeem link. |

**Per-step notes:** The referral ask replaces program 05's cross-sell for high-rating customers in the same week; never send both. Quiet hours on. Frequency caps apply except to the tier celebration, which is expected.

## Guardrails

- **Fraud controls:** Referral rewards pay out only after the referee's first purchase is delivered and outside the return window. Block self-referrals (same payment method, shipping address, or device). Cap rewards per referrer per month. Referee discount is single-use and first-order only.
- Terms are linked in every referral message.
- Points balance and expiry date come from the loyalty system of record at send time; if the lookup fails, the message aborts rather than showing a wrong number.

## Measurement

| Item | Definition |
|---|---|
| Primary KPIs | Referral: referred first purchases per 1,000 asks, treatment vs 10% holdout. Loyalty: redemption rate of expiring points vs holdout. |
| Secondary metrics | Referee 90-day repeat rate (referred customers should be as good as organic ones), reward cost per acquired customer, fraud-rejected referral share, tier-upgrade to next-purchase time |
| Control design | 10% holdout per canvas |
| Attribution window | Referral: 60 days from ask. Expiry: until the expiry date. |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, fraud rejection rate (a spike means the rules are being gamed) |
| Kill criteria | Reward cost per acquired customer above the paid-acquisition benchmark for two consecutive months |
