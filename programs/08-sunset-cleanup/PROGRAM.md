# 08 · Sunset Cleanup

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvas:** `CANVAS | Sunset Cleanup | v1`
> **Owner:** Lifecycle Manager, with the deliverability owner
> **Related programs:** Every program depends on this one. 06 Winback hands off here; 12 Re-permissioning handles engaged subscribers.

## Goal

Stop mailing addresses that have not engaged in 180 days, after one honest chance to stay, so the rest of the list reaches the inbox.

## Why it works

Mailbox providers judge a sender by how recipients react. Addresses that never open, click, or reply tell the provider your mail is unwanted, and some of them are recycled spam traps. Every send to them lowers placement for the subscribers who do want your email. Removing them usually costs almost nothing in revenue, because they were not buying from email anyway, and it often lifts engagement rates and placement for everyone who remains.

A re-permission email gives the few who still care a one-click way to stay. A final notice a week later confirms the decision. Then the address is suppressed from promotional mail. It is not deleted and not unsubscribed: the customer can still buy, still get transactional mail, and re-subscribe at any time.

## Entry

| Setting | Value |
|---|---|
| Entry type | Scheduled, weekly (Tuesday 10:00 user local time) |
| Trigger | Membership in segment `unengaged-180d` |
| Trigger properties used | None (scheduled). Profile attributes only. |
| Entry window | Always on |
| Re-eligibility | 365 days |

## Audience

**Target:** `unengaged-180d`: opted-in addresses with no opens (excluding machine opens where your data allows), no clicks, and no purchases in 180 days, and at least 180 days since signup.

**Exclusions:** Anyone who purchased in the last 180 days (they are customers even if they ignore email), `subscription-active`, `internal-test-users`.

**Exit criteria:** Clicks the stay link (alias `sunset-stay`), makes a purchase, or clicks any email.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | Day 0 | Email | `EM_SUNSET_S1` | Re-permission | "Do you still want to hear from us?" One stay button (alias `sunset-stay`), a link to reduce frequency via the preference center. |
| 2 | Day 7 | Email | `EM_SUNSET_S2` | Final notice | "This is the last one unless you tell us otherwise." Same stay button. |
| 3 | Day 14 | User Update | `UPD_SUNSET_SUPPRESS` | Suppress | Sets `promo_suppressed` = `true` and `promo_suppressed_at`. No message. |

**Per-step notes:** A second canvas or a segment filter must apply the suppression: every promotional send excludes `promo_suppressed` = `true`. Stay clicks set `promo_suppressed` = `false` and `sunset_stayed_at`. Quiet hours on. Frequency caps apply.

## Guardrails

- **Unsubscribe versus suppression policy (document it and keep it consistent):** Suppression is a sender-side decision to stop promotional mail. It does not change `email_subscribe`, does not stop transactional mail, and is reversed the moment the customer re-engages (a click, a purchase, a re-signup). Unsubscribe is the customer's decision, recorded in `email_subscribe`, and is never reversed by the sender. Never convert a suppression into an unsubscribe, and never mail a suppressed address "just this once" for a big sale.
- No guilt copy, no fake deadlines. The stay button works forever; the notice is about your sending, not their account.
- Run the sunset before any IP or domain warming, and weekly after.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Deliverability delta: inbox placement rate and open/click rates for the remaining list, 30 days before vs 30 days after the first full sunset cycle, by mailbox provider |
| Secondary metrics | Stay rate (target small; a high stay rate means the engagement definition is too strict), spam-trap hits, bounce rate, complaint rate across all programs |
| Revenue check | Revenue from the suppressed group in the 90 days before suppression should be near zero. If it is not, the definition is catching customers; loosen it. |
| Control design | 10% holdout keeps receiving normal mail for one cycle to size the revenue impact, then joins the next sunset. |
| Attribution window | 30 days after each cycle |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send |
| Kill criteria | Suppressed-group revenue materially above zero vs the holdout |
