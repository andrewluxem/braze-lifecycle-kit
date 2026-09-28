# 04 · Cart Abandonment: Build Recipe

Build time: about 75 minutes, most of it testing the Connected Content cart lookup.

## Before you start

- [ ] Custom event `added_to_cart` fires for identified users with `cart_id`, `cart_total`, and `items` (array of objects with `product_id`, `name`, `url`, `image_url`, `price`, `quantity`). Braze supports nested event properties; confirm the array appears intact on a test user's event.
- [ ] Custom event `cart_cleared` fires when a cart is emptied or expires server-side.
- [ ] Cart API endpoint `https://api.example.com/v1/carts/{cart_id}` returns JSON: `{"cart_id", "item_count", "subtotal", "subtotal_display", "margin_eligible", "free_shipping_remaining_display", "items": [{"product_id", "name", "url", "image_url", "quantity", "line_total_display", "low_stock"}]}`. It answers in under 2 seconds at p95 and returns 404 for unknown carts.
- [ ] A Connected Content credential named `northwind_cart_api` is stored in **Settings > Connected Content**. The API key lives there, never in Liquid.
- [ ] Promotion code list `northwind_cart10`: single-use codes, 10% off, the commerce platform enforces a 48-hour expiry from issue. Low-inventory alert at 20%.
- [ ] Segments exist: `discount-abusers`, `internal-test-users`, `push-enabled`.
- [ ] SMS subscription group `sms-marketing` exists, and SMS consent capture is live. Skip the SMS steps entirely if not.
- [ ] Content blocks exist: `CB_GLOBAL_HEADER`, `CB_GLOBAL_FOOTER`, `CB_CART_SERVICE_PROMISES`.

## Canvas construction

1. **Create the canvas.** Name `CANVAS | Cart Abandonment | v1`. Tags `lifecycle`, `abandonment`, `always-on`, `revenue`.

2. **Conversion events.** Primary: **Makes Purchase** (any product), deadline 7 days.

3. **Entry schedule.** **Action-Based**, trigger **Perform Custom Event** `added_to_cart`. Always on, user local time zone.

4. **Target audience.** Filters (AND):
   - Segment is not `internal-test-users`
   - **Last Purchase Date** more than 2 hours ago, OR **Number of Purchases** equals 0
   - (Email Available is `true`) OR (segment `push-enabled`)

5. **Entry controls.** **Allow re-entry** ON, after **7 days**.

6. **Exit criteria.** Exception events: **Makes Purchase** (any product); **Performs Custom Event** `cart_cleared`.

7. **Send settings.**
   - Subscription settings: opted-in users only for email (or subscribed or opted-in if program 02 is not live). SMS sends only to subscribers of `sms-marketing`.
   - Quiet hours: ON, 21:00 to 08:00 user local time, **send at next available time**.
   - Frequency capping: ON.
   - Rate limiting: set a per-minute limit your cart API can serve with headroom (each email step makes one Connected Content call per recipient). Example: 2,000 per minute for an API rated at 50 requests per second.

8. **Control group.** Variant 1 at 90%, **Control** at 10%.

9. **Qualification delay.** **Delay** step `DLY_CART_QUALIFY`: **1 hour**.

10. **Audience Paths for SMS.** Add **Audience Paths** `SPLIT_CART_SMS`:
    - Path `sms-consented`: **Subscription Group** `sms-marketing` is subscribed.
    - Everyone else.

11. **Step 1: Message (everyone else path).** Add **Message** step with two channels: Email `EM_CART_S1` and Push `PSH_CART_S1` (iOS and Android). Paste the matching blocks from [`snippets.liquid`](snippets.liquid). Push on-click: deep link from the snippet.

12. **Step 1: Message (SMS path).** On the `sms-consented` path, add the same **Message** step with three channels: `EM_CART_S1`, `PSH_CART_S1`, and SMS `SMS_CART_S1`. The SMS body aborts itself when entry `cart_total` is under 75, so only high-value carts get a text. Turn on Braze link shortening for the SMS.

13. **Rejoin and delay.** Connect both paths to one **Delay** step: **23 hours** (T+24h from entry).

14. **Step 2: Message.** **Message** step `EM_CART_S2`. Paste block `EM_CART_S2`.

15. **Delay.** **Delay** step: **48 hours** (T+72h).

16. **Decision Split.** **Decision Split** `SPLIT_CART_INCENTIVE`: segment `discount-abusers`. Yes path: variant B. No path: variant A.

17. **Step 3A: Message.** On the No path, **Message** step `EM_CART_S3A`. Paste block `EM_CART_S3A`. It fetches the live cart, checks the margin fence (`margin_eligible` and subtotal of $40 or more), and only then pulls a code from `northwind_cart10`. A cart that fails the fence renders the no-code content, and no code is consumed.

18. **Step 3B: Message.** On the Yes path, **Message** step `EM_CART_S3B`. Paste block `EM_CART_S3B`.

19. **End.** Both paths end after step 3.

20. **Priority.** In the canvas description, record: "Outranks CANVAS | Browse Abandonment | v1." If your workspace exposes canvas priority for frequency-capping ties, set this canvas above browse abandonment.

21. **Summary check.** Re-entry 7 days, 1-hour delay first, exits for purchase and `cart_cleared`, quiet hours on, control 10%, rate limit set.

22. **Seed and launch.** Seed group `seeds-lifecycle` on email steps. Save as draft, QA, launch before program 03.

## Program-specific QA additions

Run [`library/qa-checklist.md`](../../library/qa-checklist.md) first, then these:

- [ ] Add two items to a test cart, wait 1 hour. `EM_CART_S1` shows both items with live prices from the API, and the push names the first item plus "1 more."
- [ ] Add one item, then add a second item 20 minutes later. Step 1 shows both items (live cart), not only the first (entry snapshot).
- [ ] Point the canvas at a staging cart API that returns 500. Step 1 renders from entry `items`, hides per-item prices, and shows the entry `cart_total` with the "at the time you added it" label.
- [ ] Staging API returns 200 with an empty `items` array. Step aborts; no email, no push.
- [ ] Purchase within the first hour: no messages. Purchase after step 1: no steps 2 or 3. Fire `cart_cleared` after step 2: no step 3.
- [ ] SMS-consented test user with a $120 cart receives `SMS_CART_S1`; with a $40 cart, receives email and push only. A user without SMS consent never receives SMS.
- [ ] Cart abandoned at 23:00 local: step 1 arrives at 08:00 local on all channels.
- [ ] Step 3A with an eligible $60 cart: a real code, valid once, expires 48 hours after send, expiry date correct in the email. With `margin_eligible` = `false` or a $30 subtotal: no code shown, and the code list count does not drop.
- [ ] A `discount-abusers` test user receives `EM_CART_S3B` and no code.
- [ ] Low-stock line appears only for items the API flags `low_stock` = `true`.
- [ ] A cart item name containing `&` or quotes renders correctly in HTML and in the push.
- [ ] Load test: rate limit keeps Connected Content calls under the API's rated capacity (check API logs during a staged burst).
