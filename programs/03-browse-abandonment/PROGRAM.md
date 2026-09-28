# 03 · Browse Abandonment

> **Status:** ✅ Complete
> **Canvas:** `CANVAS | Browse Abandonment | v1`
> **Owner:** Lifecycle Manager
> **Related programs:** 04 Cart Abandonment (outranks this canvas), 05 Post-Purchase (takes over on purchase), 09 Back-in-Stock (handles out-of-stock views)

## Goal

Turn identified shoppers who viewed a product and left without adding to cart into buyers within 14 days, without stepping on cart abandonment.

## Why it works

A product view is a weak but specific signal: the shopper told you exactly what caught their interest. Intent decays fast, so the reminder has to arrive the same day, and it has to show the exact product, not a generic "you might like" grid. The first email works as a bookmark ("here's the thing you were looking at") rather than a sales push. The second email, a day later, answers the most common reason for leaving a product page without acting: uncertainty. Ratings, review count, and one real review do that work.

Browse abandonment is the lowest-intent program in this kit, so it must yield to anything with a stronger signal. The 4-hour qualification delay gives the shopper time to come back on their own, and gives cart abandonment (1-hour qualification) the first claim on anyone who adds to cart. Exit on `added_to_cart` hands them over cleanly. Without that ordering, a shopper who browses and then adds to cart gets two overlapping reminder streams about the same product.

No incentive. A discount on a product view trains browsers to leave pages on purpose.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Custom event `product_viewed` |
| Trigger properties used | `product_id` (string): catalog lookup key. `name` (string): headline fallback. `url` (string): product link. `image_url` (string): hero image fallback. `price` (float): entry filter only, never displayed (see Guardrails). `category` (string): sensitive-category exclusion. |
| Entry window | Always on |
| Re-eligibility | 7 days. One browse stream per user per week, no matter how many products they view. |

The canvas uses the product from the view that caused entry. A user who views five products in one session enters on the first view. If you want the most recent or most viewed product instead, maintain a `recently_viewed` array attribute in the SDK and read it at send time; the kit's snippets support the entry-product approach only.

## Audience

**Target:** Identified users (known `external_id`) with a reachable email address who view a product priced at $20 or more.

**Exclusions:**
- Email subscription status not `opted_in` (or not `subscribed`/`opted_in` if you do not run program 02).
- Users who bought the same product in the last 90 days (checked at send time by the snippet against the `purchased_product_ids_90d` array attribute).
- Users who received any cart abandonment message in the last 24 hours (checked in the pre-send Decision Split).
- `internal-test-users`.

**Exit criteria:**
- Performs custom event `added_to_cart` (cart abandonment takes over).
- Makes a purchase (any product).
- Email subscription status becomes `unsubscribed`.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | T+4h (end of qualification) | Email | `EM_BROWSE_S1` | Bookmark the product | The exact product: image, name, current price and stock from the catalog, link back. One line of copy, one button. |
| 2 | T+24h | Email | `EM_BROWSE_S2` | Resolve uncertainty | Same product with star rating, review count, one short review, returns promise. Falls back to brand-level social proof if the product has no reviews. |

**Per-step notes**

- **Qualification:** The first send happens at the end of the 4-hour qualification delay. Anyone who adds to cart or buys during those 4 hours has already exited.
- **Splits:** A Decision Split before step 1 removes users who received a cart abandonment message in the last 24 hours. The snippet itself aborts if the product is out of stock (program 09 owns that case) or was purchased since entry.
- **Quiet hours:** On, 9 p.m. to 8 a.m. user local time, send at the next allowed time. A browse reminder at 2 a.m. is noise.
- **Frequency caps:** Counts toward global caps. If the user is capped, the step does not send; that is the right outcome for the lowest-priority program.
- **Canvas priority:** Below cart abandonment. Enforced three ways: the 4-hour delay (cart qualifies at 1 hour), exit on `added_to_cart`, and the 24-hour cart-message exclusion. If your workspace offers a priority setting for canvases competing under frequency caps, rank `CANVAS | Cart Abandonment | v1` above this one.
- **Incentives:** None.

## Guardrails

- Never display the price from the event. The event price is whatever the page showed at view time and may be stale. Current price comes from the catalog; if the catalog lookup fails, the email omits the price rather than guessing.
- Out-of-stock product at send time: abort. Do not send a reminder for something the shopper cannot buy.
- No "we noticed you were looking" or "we saw you" copy. The message is useful without announcing the tracking.
- Anonymous visitors never enter; the trigger requires an identified user.
- Product categories on a sensitive-category list (health, intimate apparel, anything a shared inbox should not reveal) are excluded at the trigger property level. Maintain that list with your merchandising team.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | 14-day browse-to-purchase rate: entrants who purchase any product within 14 days / entrants. Starting target: +1 to +2 points of lift over control (a hypothesis to test, not a promise). |
| Secondary metrics | Same-product purchase rate, revenue per entrant, share of entrants who later enter cart abandonment, click-to-open rate by step |
| Control design | 10% global control at canvas entry. Control users still qualify for cart abandonment and every other program. |
| Attribution window | 14 days from entry |
| Conversion event | Primary: Makes Purchase (any product), deadline 14 days. Secondary: Performs `added_to_cart`, deadline 3 days. |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, total email sends per recipient per week (browse should not push anyone over your weekly cadence promise) |
| Kill criteria | Unsubscribe rate above 0.3% on either step for two consecutive weeks, or no lift over control after two full 14-day cycles |
