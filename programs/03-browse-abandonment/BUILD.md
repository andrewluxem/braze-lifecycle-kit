# 03 · Browse Abandonment: Build Recipe

Build time: about 40 minutes. Launch only after program 04 (cart abandonment) is live, so the priority ordering exists from day one.

## Before you start

- [ ] Custom event `product_viewed` fires for identified users only, with `product_id`, `name`, `url`, `image_url`, `price`, and `category`. Check a live test user's event log for all six.
- [ ] A Braze catalog named `products` exists, keyed on `product_id`, with at least `name`, `price`, `price_display` (preformatted string such as `$49.90`), `in_stock` (boolean), `url`, `image_url`, `rating`, `review_count`, `top_review`. It syncs at least daily; hourly is better.
- [ ] Content blocks exist: `CB_GLOBAL_HEADER`, `CB_GLOBAL_FOOTER`, `CB_BROWSE_SOCIAL_PROOF_FALLBACK`.
- [ ] Program 04 is live and its step names are final (you reference `EM_CART_S1`, `PSH_CART_S1`, `SMS_CART_S1` below).
- [ ] Array custom attribute `purchased_product_ids_90d` is maintained by your data pipeline (rolling 90 days of purchased `product_id` values). The snippet uses it to skip products the user already owns.
- [ ] You have a list of sensitive categories to exclude (can be empty, but decide on purpose).

## Canvas construction

1. **Create the canvas.** Name `CANVAS | Browse Abandonment | v1`. Tags `lifecycle`, `abandonment`, `always-on`.

2. **Conversion events.** Primary: **Makes Purchase** (any product), deadline 14 days. Secondary: **Performs Custom Event** `added_to_cart`, deadline 3 days.

3. **Entry schedule.** **Action-Based**, trigger **Perform Custom Event** `product_viewed`, with property filters:
   - `price` **greater than or equal to** `20`
   - `category` **does not equal** each sensitive category on your list (one filter per category)

   Always on, user local time zone.

4. **Target audience.** Filters (AND):
   - **Email Available** is `true`
   - **Email Subscription Status** is `opted_in` (use "is not unsubscribed" if program 02 is not live)
   - Segment is not `internal-test-users`

5. **Entry controls.** **Allow re-entry** ON, after **7 days**.

6. **Exit criteria.** Exception events: **Performs Custom Event** `added_to_cart`; **Makes Purchase** (any product); **Email Subscription Status** changes to `unsubscribed`.

7. **Send settings.**
   - Subscription settings: opted-in users only (or subscribed or opted-in, if program 02 is not live).
   - Quiet hours: ON, 21:00 to 08:00 user local time, **send at the next available time**.
   - Frequency capping: ON (global caps apply).
   - Rate limiting: none needed at typical volume.

8. **Control group.** Variant 1 at 90%, **Control** at 10%.

9. **Qualification delay.** Add **Delay** step `DLY_BROWSE_QUALIFY`: **4 hours**. This is the priority mechanism; do not shorten it below cart abandonment's 1-hour qualification plus a margin.

10. **Pre-send Decision Split.** Add **Decision Split** `SPLIT_BROWSE_CART_CONFLICT`. Yes path condition: **Received Message from Canvas Step** in `CANVAS | Cart Abandonment | v1`, steps `EM_CART_S1`, `PSH_CART_S1`, or `SMS_CART_S1`, within the last 1 day. Yes path: end (no message). No path: continue.

11. **Step 1: Message.** Add **Message** step `EM_BROWSE_S1`. Paste block `EM_BROWSE_S1` from [`snippets.liquid`](snippets.liquid). It looks up the product in the catalog, aborts if out of stock or bought in the last 90 days, and never shows the event price.

12. **Delay.** Add **Delay** step: **20 hours** (T+24h from entry).

13. **Step 2: Message.** Add **Message** step `EM_BROWSE_S2`. Paste block `EM_BROWSE_S2`. It shows the product's rating and one review from the catalog, and falls back to `CB_BROWSE_SOCIAL_PROOF_FALLBACK` if the product has fewer than 3 reviews.

14. **End.** Users leave after step 2.

15. **Priority setting.** If your workspace exposes canvas priority for frequency-capping ties, set this canvas below `CANVAS | Cart Abandonment | v1`. Document the ranking in the canvas description either way.

16. **Summary check.** Confirm: re-entry 7 days, 4-hour delay first, exits for `added_to_cart` and purchase, quiet hours on, control 10%.

17. **Seed and launch.** Seed group `seeds-lifecycle` on both email steps. Save as draft, QA, launch.

## Program-specific QA additions

Run [`library/qa-checklist.md`](../../library/qa-checklist.md) first, then these:

- [ ] Fire `product_viewed` for a test user with a real in-stock catalog `product_id`. `EM_BROWSE_S1` arrives at about T+4h with the catalog's current price, not the event price. Change the event `price` to 1.00 in a second test to prove it is never displayed.
- [ ] Fire `product_viewed` with a `product_id` that is not in the catalog. The email renders from event `name`, `url`, and `image_url`, with no price line.
- [ ] Mark a test product `in_stock` = `false` in the catalog before T+4h. Step 1 aborts.
- [ ] Fire `product_viewed`, then `added_to_cart` 30 minutes later. The user exits browse and enters cart abandonment; no browse email sends.
- [ ] Fire `product_viewed`, then purchase within 4 hours. No browse email sends.
- [ ] A user who got `EM_CART_S1` yesterday and then views a new product receives no browse email.
- [ ] Fire `product_viewed` for five products within an hour. Exactly one canvas entry, on the first product.
- [ ] Re-fire `product_viewed` on day 3: no re-entry. On day 8: re-entry.
- [ ] Fire `product_viewed` at 22:30 user local time. Step 1 holds until 08:00.
- [ ] Price below $20 and sensitive-category views do not enter.
- [ ] Product with 0 reviews: step 2 shows the brand-level fallback, never "0 reviews" or empty stars.
- [ ] The product URL in both emails carries the right product, UTM parameters, and survives `url_encode` on IDs with spaces or special characters.
