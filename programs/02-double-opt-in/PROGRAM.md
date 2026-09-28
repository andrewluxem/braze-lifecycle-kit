# 02 · Double Opt-In

> **Status:** ✅ Complete
> **Canvases:** `CANVAS | Double Opt-In Request | v1` (Canvas A) and `CANVAS | Double Opt-In Confirm | v1` (Canvas B)
> **Owner:** Lifecycle Manager, with sign-off from whoever owns privacy compliance
> **Related programs:** 01 Welcome / Onboarding (confirmed users flow into it), 08 Sunset Cleanup, 12 Re-permissioning

## Goal

Confirm that every new email subscriber owns the address and wants the mail, and convert at least 40% of checkout signups and 25% of newsletter signups into confirmed, mailable subscribers.

## Why it works

Single opt-in lists grow faster and decay faster. Typos, fake addresses, bots, and people who were auto-subscribed at checkout all land on the list, drag down engagement, and generate the complaints and spam-trap hits that damage inbox placement for everyone else. Double opt-in trades a smaller top of funnel for a list where every address is real and asked for it. Mailbox providers reward that with better placement, which lifts every other program in this kit.

The confirmation email is the one message a subscriber is actively waiting for, so it gets the highest open rate you will ever see. It has one job: one obvious button. A single reminder at 48 hours recovers people who missed it or lost it to a promotions tab, and a hard 7-day expiry keeps stale pending addresses out of your sendable audience. Two canvases instead of one keeps the confirmation logic clean: Canvas A asks, Canvas B records the answer the instant it arrives, regardless of which email the click came from.

## Entry

**Canvas A (request)**

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Custom event `email_subscribed` |
| Trigger properties used | `signup_source` (string: `checkout`, `footer`, `popup`): copy variant and reporting cut |
| Entry window | Always on |
| Re-eligibility | 7 days, so a user whose request expired can ask again, but a bot cannot trigger repeated confirmation emails to the same address |

**Canvas B (confirm)**

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Interact with Campaign / Canvas step: **clicked link alias** `confirm-subscription` in `EM_DOI_S1` or `EM_DOI_S2` |
| Trigger properties used | None |
| Entry window | Always on |
| Re-eligibility | Never. Confirmation is recorded once. |

**Hardened variant (recommended for lists with many business addresses):** corporate security scanners pre-click links in inbound mail, which can "confirm" addresses no human saw. In this variant the confirmation link opens a page on `www.example.com` with a Confirm button, and the button fires custom event `email_confirm_clicked` through your backend or SDK. Canvas B then enters on `email_confirm_clicked` instead of the alias click. Everything else stays the same.

## Audience

**Canvas A target:** Users who fire `email_subscribed` and whose email subscription status is `subscribed` (pending), not `opted_in`.

**Canvas A exclusions:** Already `opted_in` (they confirmed before), `unsubscribed`, hard-bounced, `internal-test-users`.

**Canvas A exit criteria:** Performs custom event `email_confirmed` (written by Canvas B), or email subscription status becomes `unsubscribed`.

**Canvas B target:** Anyone who clicks the confirmation alias. **Exclusions:** users already `opted_in` (a second click is a no-op). **Exit criteria:** none; it is a single-step canvas.

## Message map

| Canvas | Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|---|
| A | 1 | T+0 | Email | `EM_DOI_S1` | Request confirmation | One sentence, one button ("Confirm my subscription", alias `confirm-subscription`), what they will get, how often. No products, no promotions. |
| A | 2 | T+48h, if not confirmed | Email | `EM_DOI_S2` | Reminder | "Still want in?" plus the same button and alias. Mentions the request expires in 5 days. |
| A | 3 | T+7d, if not confirmed | User Update | `UPD_DOI_EXPIRE` | Expire | Sets `doi_status` = `expired`. No message. |
| B | 1 | On click | User Update | `UPD_DOI_CONFIRM` | Record consent | Sets `email_subscribe` = `opted_in`, `doi_status` = `confirmed`, `doi_confirmed_at`, and logs event `email_confirmed`. |

**Per-step notes**

- **Splits:** Canvas A uses a Decision Split before step 2 and step 3 on `doi_status` is not `confirmed`. Confirmed users should already have exited via the `email_confirmed` exit event; the split is a second lock.
- **Control:** Step 1 has no control group. It is a permission request the user asked for, which the measurement framework treats like a transactional message; withholding it would only produce unconfirmable addresses. The 10% control applies to step 2 through an Experiment Paths step, so the reminder's lift is measured.
- **Quiet hours:** Step 1 sends immediately at any hour; the user is waiting for it. Step 2 releases at 10 a.m. user local time.
- **Frequency caps:** Canvas A is exempt from global frequency caps. A confirmation that is capped behind a promotional send defeats the program.
- **Incentives:** None in the confirmation emails. If the signup form promised a discount, the discount arrives in program 01 after confirmation, never before.

## Guardrails

- Confirmation emails carry no promotional content. They request consent and nothing else. Your counsel should confirm the rules for each jurisdiction you mail.
- Once DOI is live, every promotional campaign and canvas must send to **opted-in users only**. If campaigns still send to "subscribed or opted in," pending and expired addresses keep receiving marketing and the program does nothing.
- Signup forms use a CAPTCHA or honeypot field, and Canvas A has an entry rate limit. Without them, bots can use your confirmation email to flood a victim's inbox (subscription bombing).
- Store `doi_confirmed_at` and `signup_source` so you can show when and where consent was given.
- Never set `opted_in` anywhere except Canvas B (or an import with verifiable prior consent). One writer for consent.
- Expired users are not re-mailed with reminders. They can re-subscribe, which re-enters them into Canvas A after the 7-day re-eligibility window.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Confirmation rate: confirmed within 7 days / entered Canvas A. Starting targets, cut by `signup_source`: at least 40% for checkout signups, at least 25% for newsletter (footer and popup) signups. These are planning targets to test against, not guarantees. |
| Secondary metrics | Time to confirm (median), share of confirmations from step 2, confirmation rate by mailbox provider (a sudden drop at one provider usually means step 1 is landing in spam there), 30-day engagement of confirmed users |
| Control design | 10% holdout on step 2 (Experiment Paths control). Step 1 is exempt. Program 01 keeps its own 10% control downstream. |
| Attribution window | 7 days from Canvas A entry |
| Conversion event | Custom event `email_confirmed`, deadline 7 days (Canvas A) |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, step 1 hard-bounce rate <3%, step 1 delivered-to-open lag (if median time to confirm climbs, check placement) |
| Kill criteria | Complaint rate above 0.1% on step 2 for two consecutive weeks: remove step 2 and rely on step 1 alone |

If checkout confirmation sits well under target, look at the checkout form before the email: a pre-checked box or unclear wording produces signups that never meant to subscribe.
