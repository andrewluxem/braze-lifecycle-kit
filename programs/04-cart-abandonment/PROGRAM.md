# 04 · Cart Abandonment

> **Status:** ✅ Complete
> **Canvas:** `CANVAS | Cart Abandonment | v1`
> **Owner:** Lifecycle Manager
> **Related programs:** 03 Browse Abandonment (ranks below this canvas), 05 Post-Purchase (takes over on purchase), 09 Back-in-Stock (handles items that go out of stock)

## Goal

Recover 10% to 15% of abandoned carts within 7 days, with incentives reserved for the last step and fenced so they protect margin.

## Why it works

An abandoned cart is the strongest intent signal short of a purchase. The shopper chose specific items, often a size and color, and got to within a few clicks of paying. Most carts are not abandoned because of price. They are interrupted (a meeting, a crying kid, a phone battery), stalled by an unanswered question (shipping cost, delivery date, return policy, fit), or parked on purpose while the shopper compares. Each step answers one of those in order.

Step 1 at one hour handles interruptions: a plain reminder with the cart contents, sent while the shopper still remembers the decision. Step 2 at 24 hours handles objections: reviews, shipping thresholds, delivery estimates, and returns, the questions a shopper cannot ask a website. Step 3 at 72 hours handles the price-sensitive remainder, and it is the only step with money on the table. Putting the discount last matters: the shoppers who were only interrupted buy at full price after step 1, so the code goes only to people who needed it. Excluding `discount-abusers` from the code keeps the program from rewarding the shoppers who abandon carts on purpose to fish for a code.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Custom event `added_to_cart` |
| Trigger properties used | `cart_id` (string): live cart lookup key. `items` (array of objects: `product_id`, `name`, `url`, `image_url`, `price`, `quantity`): fallback line items. `cart_total` (float): SMS threshold and incentive fence. |
| Entry window | Always on |
| Re-eligibility | 7 days. One recovery stream per user per week. |

The entry event captures the cart at the moment of the first add. Shoppers keep adding and removing, so every message fetches the live cart from the commerce API through Connected Content and falls back to the entry snapshot only if the API fails.

## Audience

**Target:** Identified users with a reachable email address (or push token) who add an item to cart.

**Exclusions:**
- Email subscription status not `opted_in` (or `unsubscribed`, if program 02 is not live). Push-only users still receive push.
- `internal-test-users`.
- Users who purchased in the last 2 hours (guards against a checkout that fires `added_to_cart` during an upsell flow).

**Exit criteria:**
- Makes a purchase (any product).
- Performs custom event `cart_cleared`.

Unsubscribing from email is deliberately not an exit: Braze's send-time subscription check already skips email steps for that user, and a push-enabled user can still receive the push.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | T+1h | Email + Push | `EM_CART_S1`, `PSH_CART_S1` | Reminder for interrupted shoppers | Live cart line items (image, name, quantity, price), cart total, one checkout button. Push: first item name plus count of others, deep link to cart. |
| 1 (optional) | T+1h | SMS | `SMS_CART_S1` | Reminder for high-value carts | Only with SMS marketing consent and `cart_total` of $75 or more. One line, link to cart, opt-out language. |
| 2 | T+24h | Email | `EM_CART_S2` | Answer objections | Cart recap, top review for the first item, free-shipping threshold and progress toward it, delivery estimate, 60-day returns. No discount. |
| 3 | T+72h | Email | `EM_CART_S3A` (incentive) / `EM_CART_S3B` (no incentive) | Close price-sensitive shoppers | A: single-use 10% code, 48-hour expiry, shown only when the cart passes the margin fence. B (`discount-abusers`, or carts that fail the fence): low-stock or popularity note if true, cart recap, no code. |

**Per-step notes**

- **Qualification:** The 1-hour delay before step 1 is the qualification window. Anyone who buys or clears the cart in that hour exits without a message.
- **Splits:** Audience Paths after the delay send SMS-consented users down a path that also includes `SMS_CART_S1`; the SMS body aborts itself when `cart_total` is under $75. A Decision Split on `discount-abusers` before step 3 picks variant A or B.
- **Quiet hours:** On, 21:00 to 08:00 user local time, send at next available time. This applies to all channels, and it matters most for push and SMS. A cart abandoned at 11 p.m. gets step 1 at 8 a.m.
- **Frequency caps:** Step 1 is the most valuable message in the kit; confirm your global caps do not routinely block it. Steps 2 and 3 count toward caps normally.
- **Incentives:** 10% off, single-use, from promotion code list `northwind_cart10`, expires 48 hours after send. Margin fence: live cart must return `margin_eligible` = `true` from the commerce API (no heavily discounted or low-margin items) and a subtotal of $40 or more. Carts that fail the fence get variant B content even outside `discount-abusers`. No escalation: there is never a 15% or 20% follow-up.

## Guardrails

- Every message shows the live cart. If the live cart is empty, the step aborts: an email about an empty cart is worse than no email.
- Prices shown come from the live cart API. The entry snapshot is a fallback for names and images; when the fallback is used, individual prices are hidden and only the entry `cart_total` shows, labeled as "at the time you added it."
- SMS requires explicit SMS marketing consent (subscription group `sms-marketing` = subscribed), respects quiet hours, and includes opt-out language. Operational checklist only; confirm SMS rules for your jurisdictions with counsel.
- The code's 48-hour expiry is enforced by the commerce platform, and the email states the expiry date. Never extend it after the fact.
- No "your cart will be deleted" or false scarcity. Low-stock language appears only when the catalog reports inventory under a real threshold.
- Promotion code list low-inventory alert at 20% remaining.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | 7-day cart recovery rate: entrants who purchase within 7 days / entrants. Starting target: 10% to 15% recovery for the treatment group, with lift over control reported alongside (the control also recovers some carts on its own). Targets are planning hypotheses, not guarantees. |
| Secondary metrics | Recovered revenue net of discounts, incremental revenue per entrant vs control, share of recoveries by step, code redemption rate, average discount cost per recovered order, SMS path recovery rate vs email-only |
| Control design | 10% global control at canvas entry |
| Attribution window | 7 days from entry |
| Conversion event | Primary: Makes Purchase (any product), deadline 7 days |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, SMS opt-out rate <1% per send, share of step 3 sends that are variant A (if it climbs, full-price recoveries are being subsidized) |
| Kill criteria | Complaint rate above 0.1% on any step, SMS opt-out above 2% on any send, or step 3 incremental margin negative for two consecutive monthly readouts |

Report recovery by step. If step 3 recovers a large share and steps 1 and 2 recover little, shoppers may be waiting for the code; test variant B for everyone for one cycle and compare.
