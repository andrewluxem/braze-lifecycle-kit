# 09 · Back-in-Stock + Price Drop

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvases:** `CANVAS | Back In Stock | v1` and `CANVAS | Price Drop | v1`
> **Owner:** Lifecycle Manager, with merchandising
> **Related programs:** 03 Browse Abandonment (skips out-of-stock products and leaves them here), 04 Cart Abandonment

## Goal

Tell shoppers the moment a product they asked about is available or cheaper, and convert that alert into a purchase within 7 days.

## Why it works

These are the rare messages a customer explicitly asked for, about a specific product, with a specific condition. They get the highest engagement of any promotional message because they are closer to a service than a promotion. The value depends entirely on speed and accuracy: an alert that arrives after the restock sells out, or for a price drop of 2%, spends that trust.

Back-in-stock runs on an explicit signup ("notify me"). Price drop runs on either an explicit watch or strong recent intent (a view or cart add in the last 30 days). One reminder at 48 hours catches people who saw the alert and meant to come back.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based (both canvases), triggered by an API call from the inventory or pricing service |
| Trigger | Back in stock: custom event `back_in_stock` logged per waiting user when inventory returns (the original request is `back_in_stock_signup`, property `product_id`). Price drop: custom event `price_dropped` per watching or recently interested user. |
| Trigger properties used | `product_id`, `name`, `url`, `image_url`, `available_qty` (back in stock); `old_price`, `new_price`, `drop_pct` (price drop) |
| Entry window | Always on |
| Re-eligibility | Per product: 30 days. A user can get alerts for different products in the same week. |

## Audience

**Target:** Users who signed up for the specific product (back in stock) or watched or viewed or carted it in the last 30 days (price drop), with the channel consent for each channel used.

**Exclusions:** Already purchased the product since signing up; `internal-test-users`.

**Exit criteria:** Purchases the product; the product goes out of stock again (event `out_of_stock`, which ends back-in-stock paths).

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | Immediately | Email + Push | `EM_BIS_S1`, `PSH_BIS_S1` / `EM_PRICEDROP_S1`, `PSH_PRICEDROP_S1` | Alert | The product, current price from the catalog, one button. Price drop shows old and new price. |
| 2 | T+48h | Email | `EM_BIS_S2` / `EM_PRICEDROP_S2` | Reminder | Same product, stock or price re-checked at send time. |

**Per-step notes:** Quiet hours on for push; email may send at any hour (the alert loses value with every hour of delay). Both steps re-check the catalog and abort if the condition no longer holds.

## Guardrails

- **Stock threshold:** The inventory service fires `back_in_stock` only when available quantity covers the waiting list plus a buffer (for example, at least 20% of the number of waiting users, minimum 10 units). Do not notify 5,000 people about 12 units. When stock is short, notify in batches, earliest signups first.
- **Price-drop minimum:** Fire `price_dropped` only when the drop is 10% or more and the new price has been live for at least 1 hour (so a pricing error that gets reverted does not trigger alerts).
- Step 2 aborts if the product is out of stock or the price has gone back up.
- Never show a price from the event in step 2; read the catalog at send time.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Notify-to-purchase rate: entrants who buy the alerted product within 7 days / entrants, treatment vs 10% holdout |
| Secondary metrics | Time from alert to purchase, sell-through of restocked inventory, share of alerts aborted for stock (a high share means the threshold is too low) |
| Control design | 10% holdout at entry (price drop). For back in stock, where the user explicitly asked, consider a 5% holdout for the first cycle only, then remove it; document the choice. |
| Attribution window | 7 days |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, push opt-out rate per send |
| Kill criteria | Alerts landing after sell-out more than 5% of the time |
