# 07 · Churn Prevention

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvas:** `CANVAS | Churn Prevention | v1`
> **Owner:** Lifecycle Manager, with the subscription product owner
> **Related programs:** 06 Winback (for non-subscription customers), 12 Re-permissioning

## Goal

Save a measurable share of subscribers who start to cancel, and bring back a share of those who finish cancelling, without keeping anyone who wants to leave.

## Why it works

Cancellation intent is not one thing. Some subscribers have too much product, some have a price problem, some are leaving for a season, and some are done. A single "please stay" discount treats all of them as the price problem and pays the ones who would have stayed anyway.

The sequence matches the offer to the reason. At the moment of intent, an in-app message asks why and offers the matching fix: skip or pause for too-much-product, a smaller plan for price, a pause for seasonal. After the fact, an email repeats the pause option for anyone who cancelled in a hurry. A post-cancel return offer weeks later catches the people whose reason has changed. Keeping the cancel button easy to find matters: a save flow that feels like a trap produces complaints and chargebacks, and in some jurisdictions it creates legal exposure.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Custom event `subscription_cancel_initiated` |
| Trigger properties used | `subscription_id` (string), `plan` (string), `cancel_reason` (string, if the form asks before the in-app step), `tenure_months` (integer) |
| Entry window | Always on |
| Re-eligibility | 90 days |

## Audience

**Target:** Users in `subscription-active` who start a cancellation.

**Exclusions:** Accounts with a failed payment in progress (dunning owns them), accounts flagged for fraud or chargeback, `internal-test-users`.

**Exit criteria:** Custom event `subscription_saved` (paused, downgraded, or cancelled the cancel) exits the save path. Custom event `subscription_reactivated` exits the post-cancel path.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | At intent (in session) | In-App | `IAM_CHURN_S1` | Save offer matched to reason | Reason picker, then pause, skip, or downgrade options. Cancel remains one tap away. |
| 2 | T+1d if cancelled | Email | `EM_CHURN_S2` | Pause option | "Pause instead?" with a one-click pause link valid 14 days. Confirms the cancellation terms plainly. |
| 3 | T+30d after cancel | Email | `EM_CHURN_S3` | Return offer | What has changed since they left, one fenced return offer (first box at a set discount, single-use, margin-checked). |

**Per-step notes:** Audience Paths after step 1 route saved users out and cancelled users to step 2. Step 3 excludes `discount-abusers` from the offer and anyone who cancelled for a product-quality reason logged in `cancel_reason` (send them a service follow-up instead). Quiet hours on for email. Frequency caps apply to steps 2 and 3.

## Guardrails

- Cancellation always stays easy. No hidden buttons, no forced calls, no multi-page mazes. Your counsel should review the flow against the rules for each market you sell in.
- The cancellation confirmation itself is transactional and sends outside this canvas.
- One save offer per intent. No repeated offers inside the same session.
- Return offer is single-use, margin-fenced, and never better than the new-subscriber offer.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Save rate: entrants who end the flow still subscribed (active or paused) / entrants, treatment vs 10% holdout |
| Secondary metrics | 90-day re-churn of saved subscribers (a save that cancels 30 days later is not a save), pause-to-resume rate, post-cancel reactivation rate, revenue retained net of discounts |
| Control design | 10% holdout at entry sees the standard cancel flow with no save offer |
| Attribution window | 7 days for save; 90 days for re-churn and reactivation |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, support contacts mentioning "can't cancel" (target zero), chargeback rate vs control |
| Kill criteria | Any rise in "can't cancel" contacts or chargebacks vs control |
