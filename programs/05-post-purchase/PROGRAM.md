# 05 · Post-Purchase

> **Status:** ✅ Complete
> **Canvas:** `CANVAS | Post-Purchase | v1`, plus the separate transactional campaign `TRX_ORDER_CONFIRMATION`
> **Owner:** Lifecycle Manager (canvas); Ecommerce Operations (order confirmation)
> **Related programs:** 01, 03, 04 (all exit on purchase and hand off here), 09 Back-in-Stock, 10 Loyalty / Referral (referral ask at T+21d must not collide with step 3)

## Goal

Turn first and occasional buyers into repeat buyers within 90 days, and collect product reviews from a steady share of delivered orders.

## Why it works

The period right after a purchase has the highest trust and the highest anxiety of the whole relationship. The customer just gave you money and does not have the product yet. Messages that reduce anxiety (what happens next, how to get the most out of the product) earn attention that promotional email never gets, and they lower "where is my order" contacts and early returns.

Reviews depend on timing. Ask before the product arrives and you get annoyance; ask a month later and the moment has passed. Tying the ask to confirmed delivery plus five days, long enough to use the product and short enough to still care, catches the peak. Skipping anyone with an open return or support ticket keeps the ask from landing on an unhappy customer at the worst moment.

The second purchase is the hardest one to get and the best predictor of long-term value. Cross-sell at three weeks rides the afterglow of a product that has now proven itself. At 45 days the canvas splits by what was bought: consumables get a replenishment reminder close to when they run out, and durables get a nudge toward a complementary category. Neither step discounts. A customer who just paid full price should not learn that waiting pays.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Makes Purchase (any product) |
| Trigger properties used | `order_id` (string): dedupe and review links. `items` (array of objects: `product_id`, `name`, `url`, `image_url`, `category`, `is_consumable`): review links, cross-sell exclusions, step 4 split. `order_total` (float): reporting. `has_consumable` (boolean): step 4 split. `ordered_at` (time): replenishment guard. |
| Entry window | Always on |
| Re-eligibility | 14 days. Braze logs one purchase event per product, so a three-item order produces three events; the order-level properties ride on every line so whichever line triggers entry carries the full order, and the re-eligibility window keeps the user from entering three times. |

## Audience

**Target:** Every purchaser with a reachable email address.

**Exclusions:**
- Email subscription status not `opted_in` (marketing steps). The transactional confirmation still sends.
- `internal-test-users`.
- Orders flagged `is_b2b_bulk` = `true` in purchase properties (bulk orders get account-manager follow-up, not lifecycle email).

**Exit criteria:**
- Performs custom event `return_initiated`: the customer is unhappy or the product did not work. Stop selling to them in this canvas; customer service owns the next touch.
- Email subscription status becomes `unsubscribed`.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 0 | Immediately, outside the canvas | Email | `TRX_ORDER_CONFIRMATION` | Confirm the order | Separate transactional triggered campaign. Order number, items, totals, shipping address, delivery estimate. Bypasses promotional caps, no control group, no marketing content. |
| 1 | T+2d | Email | `EM_POSTPURCH_S1` | Reduce anxiety, drive product success | Shipping status language (arrived vs on its way), per-category setup or care guide, support link. No selling. |
| 2 | `delivery_confirmed` + 5d, else T+10d | Email | `EM_POSTPURCH_S2` | Review request | One review deep link per item (up to 4), star-tap entry, what reviews help with. Skipped if a return or support ticket is open. |
| 3 | About T+21d (11 days after the review step) | Email | `EM_POSTPURCH_S3` | Cross-sell | Up to 3 recommendations from the recommendations API via Connected Content, excluding anything purchased. Static bestseller fallback if the API fails. |
| 4 | About T+45d (24 days after step 3) | Email | `EM_POSTPURCH_S4A` (consumable) / `EM_POSTPURCH_S4B` (durable) | Second purchase | A: replenishment reminder with one-click reorder link for consumable items. B: complementary category nudge. Skipped if the customer has already reordered. |

**Per-step notes**

- **Splits:** After step 1, a Decision Split checks whether delivery was already confirmed (fast shipping). Delivered users wait 3 more days; everyone else enters Action Paths that wait up to 8 days for `delivery_confirmed`, then wait 5 days, or time out at T+10d. All three routes pass through a Decision Split that skips the review ask when `has_open_return` or `has_open_support_ticket` is `true`. Step 4 is a Decision Split on `last_order_has_consumable`, which a User Update step copies from the entry property at the start of the canvas (splits read profile data, not entry properties). Steps 3 and 4 use fixed delays after the review step, so their exact day varies by a few days with delivery speed; the spacing guarantees at least four days between any two canvas emails.
- **Quiet hours:** On, 21:00 to 08:00 user local time, send at next available time. Nothing in this canvas is urgent.
- **Frequency caps:** Steps 1 through 4 count toward global caps. `TRX_ORDER_CONFIRMATION` is exempt and does not count.
- **Collisions:** Program 10's referral ask also lands near T+21d. Run them as a Decision Split in this canvas (referral ask for customers with a review of 4 stars or more, cross-sell for everyone else) rather than two independent sends in the same week.
- **Incentives:** None. The review ask never offers a reward in exchange for a review; incentivized reviews have to be disclosed and they distort ratings.

## Guardrails

- The order confirmation is transactional: no promotions, no cross-sell modules, no control group. Classify it as transactional in Braze and keep it on the transactional sending path.
- Review requests go only to delivered or presumed-delivered orders, never while a return or support ticket is open, and never with an incentive.
- Cross-sell excludes every product in the order, and the snippet re-checks the recommendations against the order items in case the API ignores the exclusion.
- Replenishment copy only for items flagged consumable in the catalog. Nobody needs a reminder to rebuy a tent.
- No "we noticed" language about order contents in subject lines; subject lines can be read by anyone who sees the inbox.

## Measurement

| Item | Definition |
|---|---|
| Primary KPIs | (1) 90-day repeat purchase rate: entrants with a second order within 90 days of entry / entrants. (2) Reviews per 100 delivered orders. Starting targets: +2 to +3 points of repeat-purchase lift over control, and at least 5 reviews per 100 delivered orders. Both are planning hypotheses. |
| Secondary metrics | Revenue per entrant at 90 days, cross-sell click-through, replenishment reorder rate within 14 days of step 4A, "where is my order" contact rate vs control, return rate vs control |
| Control design | 10% global control at canvas entry. Control users still receive `TRX_ORDER_CONFIRMATION`. |
| Attribution window | 90 days from entry for repeat purchase; 30 days from step 2 send for reviews |
| Conversion events | Primary: Makes Purchase, deadline 90 days. Secondary: Performs `review_submitted`, deadline 30 days. |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, share of review asks sent to customers who later initiate a return within 7 days (should be near zero), confirmation delivery latency (p95 under 5 minutes) |
| Kill criteria | Complaint rate above 0.1% on any step, or review asks correlating with a rise in negative reviews versus control-period baseline |

Because the canvas runs 45 days and the primary window is 90, read results only on cohorts that have completed the full 90 days. Early readouts overstate or understate the effect depending on seasonality.
