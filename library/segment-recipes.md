# Segment Recipes

Reusable segment definitions for every program in the kit. Build each once, name it exactly as shown ([`naming-taxonomy.md`](naming-taxonomy.md)), and reference it from canvases instead of re-creating filters inline. A segment that three canvases share gets fixed once.

How to read the definitions:

- **Filters** are Braze segment filters joined with AND unless marked OR. Filter labels follow the Segments editor; exact wording can shift between Braze releases.
- **Data needed** lists attributes or events that must exist before the segment works. Several segments need values Braze cannot compute from its own filters (ratios, percentiles). Those come from your warehouse through Cloud Data Ingestion or `/users/track`, or from a SQL Segment Extension.
- **Refresh**: segments evaluate at send time, so filters with relative windows stay current automatically. Warehouse-fed attributes are only as fresh as their sync; sync them at least daily.
- Every segment also excludes internal accounts through `internal-test-users` at the canvas level, not inside each segment.

---

### `email-engaged-90d`

Subscribers who have shown real email engagement in the last 90 days.

- **Email Subscription Status** is `opted_in` (or is not `unsubscribed`, without program 02)
- **Last Engaged With Message** (email, click) less than 90 days ago
- OR **Last Engaged With Message** (email, open) less than 90 days ago

**Notes:** Machine opens from privacy-protected mail clients inflate opens. If your workspace distinguishes machine opens, use non-machine opens only; otherwise weight toward clicks and tighten to 60 days for open-only engagement. **Used by:** 12, deliverability warming.

### `unengaged-180d`

Opted-in addresses that have gone silent and are hurting deliverability.

- **Email Subscription Status** is `opted_in` (or not `unsubscribed`)
- **Last Engaged With Message** (email, any engagement) more than 180 days ago, OR never engaged
- **First Used App / Signup date** (custom attribute `signed_up_at`) more than 180 days ago
- **Last Purchase Date** more than 180 days ago, OR **Number of Purchases** equals 0

**Notes:** The signup-age filter keeps brand-new subscribers from qualifying before they have had a chance to engage. **Used by:** 08, excluded from 06.

### `push-enabled`

Users who can receive a push notification right now.

- **Foreground Push Enabled** is `true` (any app)

**Notes:** Use the per-app filter if you run several apps. Background push (silent) does not count. **Used by:** 01, 04, 06.

### `purchased-last-30d`

- **Last Purchase Date** less than 30 days ago

**Used by:** exclusions in promotional sends, 03 reporting.

### `first-time-buyer-90d`

Customers who made exactly one purchase, recently.

- **Number of Purchases** equals `1` (orders, not line items; see note)
- **First Purchase Date** less than 90 days ago

**Data needed:** Braze counts one purchase per product. If you log line items, the native count overstates orders. Maintain `lifetime_orders` (integer custom attribute) from your order system and filter `lifetime_orders` equals 1 instead. **Used by:** 05 reporting, 10.

### `repeat-buyer`

- `lifetime_orders` greater than or equal to `2` (or **Number of Purchases** at least 2 if you log one purchase per order)

**Used by:** 10, 11, reporting.

### `lapsed-90-180d`

Customers whose last purchase fell 90 to 180 days ago.

- **Last Purchase Date** more than 90 days ago
- **Last Purchase Date** less than 180 days ago
- **Email Subscription Status** is not `unsubscribed`

**Notes:** Tune the window to your purchase cycle. For a brand where the median time between orders is 45 days, 90 days is lapsed; for a brand selling annual gear, it may not be. Use your own repeat-purchase curve: lapsed starts around the 75th percentile of time between orders. **Used by:** 06.

### `high-intent-no-purchase-14d`

Shoppers showing strong buying signals without converting.

- **Performed Custom Event** `added_to_cart` at least once in the last 14 days, OR **Performed Custom Event** `product_viewed` at least 3 times in the last 14 days (the **X Custom Event in Y Days** filter)
- **Last Purchase Date** more than 14 days ago, OR **Number of Purchases** equals 0

**Used by:** 09 (price-drop audience), paid-media suppression and lookalike seeds, campaign planning.

### `discount-abusers`

Customers whose buying pattern shows they wait for or hunt codes.

**Definition:** at least 3 orders in the last 180 days AND at least 80% of those orders used a promotion code.

Braze segment filters cannot compute a ratio, so build it one of two ways:

**Option A (recommended): warehouse attribute.** Compute nightly in your warehouse and sync two custom attributes: `orders_180d` (integer) and `promo_order_share_180d` (float, 0 to 1). Then:

- `orders_180d` greater than or equal to `3`
- `promo_order_share_180d` greater than or equal to `0.8`

**Option B: SQL Segment Extension** on Braze's own purchase data. Braze logs a row per purchased product, so count distinct orders by an `order_id` stored in purchase properties. Column and table names follow Braze's SQL segment extension schema; confirm them in your workspace's SQL editor reference.

```sql
SELECT user_id
FROM (
  SELECT
    user_id,
    COUNT(DISTINCT PARSE_JSON(properties):order_id::string) AS orders,
    COUNT(DISTINCT CASE
      WHEN PARSE_JSON(properties):promo_code::string IS NOT NULL
      THEN PARSE_JSON(properties):order_id::string END) AS promo_orders
  FROM USERS_BEHAVIORS_PURCHASE_SHARED
  WHERE time > DATE_PART('epoch_second', DATEADD(day, -180, CURRENT_TIMESTAMP()))
  GROUP BY user_id
)
WHERE orders >= 3
  AND promo_orders / orders >= 0.8
```

Refresh the extension daily. **Used by:** 01, 04, 06, 07, 10, 11 (always as a routing split or exclusion on incentive steps, never as a full canvas exclusion, so these customers still receive non-incentive messages).

### `vip-top-5pct`

Top 5% of customers by trailing 12-month net revenue.

**Data needed:** Braze has no percentile filter. Compute `ltv_12m_percentile` (integer 1 to 100) in the warehouse and sync it nightly, then:

- `ltv_12m_percentile` greater than or equal to `96`

Alternative: a SQL Segment Extension with `NTILE(100) OVER (ORDER BY SUM(price * quantity))` over the last 365 days. Use net revenue (after returns) if your purchase data carries returns; gross revenue over-rewards heavy returners. **Used by:** exclusions from incentive steps (VIPs rarely need a code), early-access sends, 10.

### `subscription-active`

- Custom attribute `subscription_status` is `active` OR `paused`

**Data needed:** `subscription_status` from your billing system, updated on every state change (event-driven, not nightly). **Used by:** 07, excluded from 06.

### `subscription-at-risk`

Active subscribers likely to cancel soon.

- In `subscription-active`
- AND one or more of: custom attribute `churn_risk_score` greater than or equal to `0.7` (from your model, or a Braze predictive churn segment if you use one), skipped 2 or more of the last 3 deliveries (`skips_last_3`), failed payment in the last 30 days, **Last Used App** more than 30 days ago

**Notes:** Use this segment for proactive retention (value reminders, plan-fit suggestions), not for discounting. Program 07 triggers on explicit cancel intent; this segment is for the quiet majority who drift first. **Used by:** 07 (proactive extension), reporting.

---

## Utility segments referenced in builds

- `internal-test-users`: email address ends with your internal test domain (Northwind uses `@test.example.com`), OR custom attribute `is_internal` is `true`. Excluded from every canvas; seed groups handle internal visibility.
- `seeds-lifecycle`: a Braze seed group (not a segment) containing inbox-placement seed addresses and the lifecycle team, added to email steps.
