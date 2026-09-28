# 11 · Anniversary / Milestone

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvas:** `CANVAS | Milestones | v1`
> **Owner:** Lifecycle Manager, with brand
> **Related programs:** 10 Loyalty / Referral

## Goal

Mark a customer's birthday, one-year anniversary, and 10th purchase with one warm message and a small gift, and measure whether it improves how customers feel about the brand.

## Why it works

Most lifecycle email asks for something. A milestone message gives something, with no condition attached beyond a reasonable window to use it. That reciprocity is the point. It is also one of the few messages a customer might forward or mention to a friend. The gift stays small (free shipping, a small credit, an accessory) because a large one turns recognition into a transaction and trains customers to wait for their birthday.

This program is judged on sentiment, not revenue. If it is measured on revenue, it will slowly turn into a birthday coupon blast, and the warmth that made it work will go.

## Entry

| Setting | Value |
|---|---|
| Entry type | Scheduled, daily at 09:00 user local time |
| Trigger | Membership in any of: birthday today (`date_of_birth` anniversary filter), first order exactly 1 year ago (custom attribute `first_order_at` anniversary), 10th order reached (custom attribute `lifetime_orders` = 10 and `milestone_10_sent` not true) |
| Trigger properties used | None (scheduled). Profile attributes only. |
| Entry window | Always on |
| Re-eligibility | 360 days (birthday and anniversary recur yearly; the 10th-order milestone is guarded by `milestone_10_sent`) |

## Audience

**Target:** Opted-in customers who hit a milestone today.

**Exclusions:** Open support ticket or return in the last 14 days (a celebration on a bad week reads as tone-deaf), `internal-test-users`. If two milestones fall on the same day, send one combined message.

**Exit criteria:** None needed; one message per entry.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | Day of milestone | Email (+ In-App for app users) | `EM_MILESTONE_S1`, `IAM_MILESTONE_S1` | Recognize | Milestone-specific copy (Audience Paths: birthday, anniversary, 10th order), a small gift valid 30 days, no other offers or product grids. |

**Per-step notes:** Audience Paths pick the variant; a User Update step sets `milestone_10_sent` = `true` after the 10th-order message. Quiet hours on. Frequency caps apply. `discount-abusers` receive a non-monetary gift (early access, a note from the team) instead of a credit.

## Guardrails

- One message per milestone. No reminder, no "your gift expires tomorrow" chaser.
- Birthdays are collected only with a clear reason given at collection, and only month and day are needed. Never ask for or display birth year.
- Gift is single-use, 30-day window, small enough that nobody would wait for it.
- Copy stays about the customer, not the catalog.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Sentiment: reply rate and one-question CSAT or thumbs rating embedded in the email, treatment vs holdout on a follow-up survey sent to a sample of both groups 14 days later |
| Secondary metrics | Gift redemption rate, 90-day retention of recipients vs holdout (read as a secondary signal, not the goal), forward/share clicks |
| Control design | 10% holdout |
| Attribution window | 30 days (gift window) |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send; a milestone message should run well under both |
| Kill criteria | Sentiment no better than holdout for two quarters: rework the gift and copy before cutting |
