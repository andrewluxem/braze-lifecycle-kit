# 05 · Post-Purchase: Build Recipe

Build time: about 90 minutes: 20 for the transactional campaign, 70 for the canvas.

## Before you start

- [ ] Purchases are logged to Braze (SDK or `/users/track` `purchases` object) with properties on every line: `order_id`, `items` (full order array: `product_id`, `name`, `url`, `image_url`, `category`, `is_consumable`), `order_total`, `has_consumable`, `ordered_at`, `is_b2b_bulk`.
- [ ] Custom events fire: `delivery_confirmed` (property `order_id`), `return_initiated` (property `order_id`), `review_submitted` (properties `product_id`, `rating`).
- [ ] Custom attributes are maintained: `last_delivery_confirmed_at` (time), `has_open_return` (boolean), `has_open_support_ticket` (boolean, from the helpdesk integration), `last_order_at` (time).
- [ ] Recommendations API `https://api.example.com/v1/recommendations` accepts `user`, `exclude` (comma-separated product IDs), and `limit`, and returns `{"products": [{"product_id", "name", "url", "image_url", "price_display"}]}`. Connected Content credential `northwind_recs_api` stored.
- [ ] Catalog `products` has `care_guide_url` and `category_display` for each product.
- [ ] Content blocks exist: `CB_GLOBAL_HEADER`, `CB_GLOBAL_FOOTER`, `CB_TRX_HEADER`, `CB_TRX_FOOTER`, `CB_POSTPURCH_SUPPORT`, `CB_POSTPURCH_CROSSSELL_FALLBACK`, `CB_POSTPURCH_NEXT_CATEGORY_FALLBACK`, and one `CB_POSTPURCH_NEXT_<CATEGORY>` block per top-level category you name in `EM_POSTPURCH_S4B` (Northwind uses `CAMPING`, `CLIMBING`, `APPAREL`).

## Part 1: Transactional order confirmation (separate campaign)

1. **Create the campaign.** **Messaging > Campaigns > Create Campaign > Email**. Name `TRX_ORDER_CONFIRMATION`. Tags `transactional`, `order`.

2. **Delivery.** **API-Triggered** delivery (preferred: your order system calls `/campaigns/trigger/send` with the order payload as `trigger_properties`). If you must use action-based delivery on Makes Purchase, remember it fires per line item; dedupe with re-eligibility of 1 hour or a custom `order_placed` event.

3. **Audience.** All users. No segment filters beyond `Email Available`.

4. **Subscription settings.** Send to **all users including unsubscribed** (transactional). Use your workspace's transactional email configuration or dedicated transactional IP pool if you have one.

5. **Frequency capping.** OFF. **Control group:** none. **Quiet hours:** OFF.

6. **Message.** Paste block `TRX_ORDER_CONFIRMATION` from [`snippets.liquid`](snippets.liquid). API trigger properties are referenced as `{{api_trigger_properties.${name}}}`.

7. **Launch** after QA with a real test order.

## Part 2: Canvas `CANVAS | Post-Purchase | v1`

8. **Create the canvas.** Name `CANVAS | Post-Purchase | v1`. Tags `lifecycle`, `post-purchase`, `always-on`, `retention`.

9. **Conversion events.** Primary: **Makes Purchase**, deadline 90 days. Secondary: **Performs Custom Event** `review_submitted`, deadline 30 days.

10. **Entry schedule.** **Action-Based**, trigger **Makes Purchase** (any product). Add property filter `is_b2b_bulk` **does not equal** `true`. Always on, user local time zone.

11. **Target audience.** Filters (AND): **Email Available** is `true`; **Email Subscription Status** is `opted_in` (or not `unsubscribed` without program 02); segment is not `internal-test-users`.

12. **Entry controls.** **Allow re-entry** ON, after **14 days**.

13. **Exit criteria.** Exception events: **Performs Custom Event** `return_initiated`; **Email Subscription Status** changes to `unsubscribed`.

14. **Send settings.** Opted-in only; quiet hours ON 21:00 to 08:00, send at next available time; frequency capping ON; rate limit on step 3 matched to the recommendations API's capacity.

15. **Control group.** Variant 1 at 90%, **Control** at 10%.

16. **User Update: order flags.** First step for everyone in Variant 1: **User Update** step `UPD_POSTPURCH_FLAGS`, Advanced JSON, block `UPD_POSTPURCH_FLAGS` from [`snippets.liquid`](snippets.liquid). It copies `has_consumable` and `ordered_at` from the entry properties onto the profile (`last_order_has_consumable`, `postpurch_entry_order_at`), because Decision Splits filter on profile data, not on entry properties.

17. **Delay.** **Delay** step: 2 days, release at 10:00 user local time.

18. **Step 1: Message.** **Message** step `EM_POSTPURCH_S1`. Paste block `EM_POSTPURCH_S1`.

19. **Decision Split: delivered already?** **Decision Split** `SPLIT_POSTPURCH_DELIVERED_EARLY`: custom attribute `last_delivery_confirmed_at` **more recent than** 3 days ago. Yes: step 20. No: step 21.

20. **Early-delivery delay.** On Yes: **Delay** step 4 days (review lands around T+6d, four days after step 1). Connect to step 23.

21. **Action Paths: wait for delivery.** On No: **Action Paths** step `ACT_POSTPURCH_DELIVERY`. Evaluation window **8 days** (ends T+10d). Action group `delivered`: **Performs Custom Event** `delivery_confirmed`. Everyone else advances when the window closes, straight to step 23.

22. **Delivered path delay.** On the `delivered` path: **Delay** step 5 days. Connect to step 23.

23. **Decision Split: safe to ask?** Merge all three routes into **Decision Split** `SPLIT_POSTPURCH_REVIEW_SAFE`: custom attribute `has_open_return` is `true` OR `has_open_support_ticket` is `true`. Yes: skip to step 25. No: step 24.

24. **Step 2: Message.** **Message** step `EM_POSTPURCH_S2`. Paste block `EM_POSTPURCH_S2` (per-item review links).

25. **Delay to cross-sell.** Merge both branches into one **Delay** step: **11 days**, release at 10:00 user local time. On the timeout route this puts step 3 at T+21d; on faster routes it lands between T+17d and T+24d, and it always keeps at least 11 days after the review ask. If your workspace supports personalized delays (delay until a time stored on the profile), you can instead delay until `postpurch_entry_order_at` + 21 days; keep the fixed delay if in doubt.

26. **Step 3: Message.** **Message** step `EM_POSTPURCH_S3`. Paste block `EM_POSTPURCH_S3` (Connected Content cross-sell, static fallback). If you run program 10, add a **Decision Split** before this step on custom attribute `last_review_rating` of 4 or more: Yes gets program 10's referral ask, No gets cross-sell.

27. **Delay.** **Delay** step: **24 days**, release at 10:00 user local time (step 4 lands around T+45d).

28. **Decision Split: consumable?** **Decision Split** `SPLIT_POSTPURCH_CONSUMABLE`: custom attribute `last_order_has_consumable` is `true`.

29. **Step 4A: Message.** On Yes: **Message** step `EM_POSTPURCH_S4A` (replenishment). It aborts if `last_order_at` is later than `postpurch_entry_order_at` plus one day (the customer already reordered).

30. **Step 4B: Message.** On No: **Message** step `EM_POSTPURCH_S4B` (next category). End after steps 4A and 4B.

31. **Summary check.** Re-entry 14 days, exit on `return_initiated`, control 10%, quiet hours on, conversion events set, `TRX_ORDER_CONFIRMATION` live and separate.

32. **Seed and launch.** Seed group `seeds-lifecycle`. Save as draft, QA, launch.

## Program-specific QA additions

Run [`library/qa-checklist.md`](../../library/qa-checklist.md) first, then these:

- [ ] Place a real test order with three items. `TRX_ORDER_CONFIRMATION` arrives in under 5 minutes, lists all three items, and has no promotional content. It arrives for an unsubscribed test user too.
- [ ] The same three-item order enters the canvas once (check the user's Canvas entries), and the entry properties carry all three items.
- [ ] Control-group test user receives the confirmation and nothing else.
- [ ] Fire `delivery_confirmed` on day 4. Review ask arrives on day 9. With no delivery event, the ask arrives on day 10. With delivery confirmed before day 2, the ask arrives around day 6.
- [ ] Set `has_open_support_ticket` = `true` before the review step. No review ask; the user continues to step 3.
- [ ] Fire `return_initiated` at any point. The user exits and receives nothing further.
- [ ] Review ask shows one link per item (maximum 4), each carrying the right `product_id` and `order_id`.
- [ ] Recommendations API returns a product that was in the order: the snippet filters it out. API returns 500 or times out: the static fallback block renders. API returns zero products: fallback renders.
- [ ] Order with a consumable: step 4A. Order with no consumables: step 4B. Customer who reordered on day 30: step 4A aborts.
- [ ] No two canvas emails land within 4 days of each other on any route (check the early-delivery route especially).
- [ ] B2B bulk test order does not enter the canvas but does get the confirmation.
