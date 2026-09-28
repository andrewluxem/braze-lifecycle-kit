# 01 · Welcome / Onboarding

> **Status:** ✅ Complete
> **Canvas:** `CANVAS | Welcome Series | v1`
> **Owner:** Lifecycle Manager
> **Related programs:** 02 Double Opt-In (feeds this canvas), 04 Cart Abandonment (can run in parallel), 05 Post-Purchase (takes over on first purchase)

## Goal

Convert new subscribers and account signups into first-time buyers within 30 days, without training them to wait for a discount.

## Why it works

A new subscriber is at peak attention and lowest trust. They just told you they are interested, so open rates on the first message are the highest you will ever see from them, but they have no evidence yet that you are worth buying from. The series spends that attention on trust in order: set expectations and give control (preference center), show what makes the brand different (value props), let other customers vouch for it (social proof), and only then offer a reason to act now (incentive).

The incentive comes on day 7, not day 0. Leading with a code teaches every subscriber that signing up is a coupon transaction, and it gives away margin on people who would have bought anyway. By day 7 the people with strong intent have often already purchased at full price and exited. The code goes to the remainder, who need the push. The day-14 expiry nudge uses loss aversion: a code with a deadline outperforms a code without one, and the deadline has to be real.

Exiting on purchase at any step matters as much as the messages. Nothing erodes trust faster than a "still thinking about it?" email the day after someone bought.

## Entry

| Setting | Value |
|---|---|
| Entry type | Action-based |
| Trigger | Custom event `signed_up` OR custom event `email_subscribed` |
| Trigger properties used | `signup_source` (string: `footer`, `checkout`, `popup`, `account`): tailors step 1 copy. `preferred_category` (string, optional): picks the hero block in step 3. |
| Entry window | Always on, starting at launch |
| Re-eligibility | Never. A user enters this canvas once in their lifetime. |

If you run program 02 (double opt-in), the trigger becomes `email_confirmed` (fired by program 02's Canvas B) instead of `email_subscribed`, so unconfirmed addresses never receive the series.

## Audience

**Target:** Users who fire either trigger and have a reachable email address with `email_subscribe` of `subscribed` or `opted_in`.

**Exclusions:**
- Anyone who has ever purchased (`Number of Purchases > 0`): an existing customer who creates an account is not "new." Send them to program 05 logic instead.
- Email subscription status `unsubscribed` or hard-bounced addresses.
- Internal and test accounts (segment `internal-test-users`).
- `discount-abusers` are NOT excluded from the canvas. They enter and receive the no-code variant at step 4.

**Exit criteria:**
- Performs `purchase` (any product): exits immediately, program 05 takes over.
- Email subscription status becomes `unsubscribed`: exits.

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | T+0 (immediately) | Email | `EM_WELCOME_S1` | Set expectations, give control | Thank-you, what they will get and how often, preference center link, one soft entry point to shop by category. No discount. |
| 2 | T+1d | Push (if push-enabled) or In-App | `PSH_WELCOME_S2` / `IAM_WELCOME_S2` | Reinforce value props | Three short proof points: free returns for 60 days, same-day dispatch before 2 p.m., price-match guarantee. Deep link to the category they browsed or bestsellers. |
| 3 | T+3d | Email | `EM_WELCOME_S3` | Social proof | Three bestsellers with star ratings and one customer quote each, review count, "how it's made" link. |
| 4 | T+7d | Email | `EM_WELCOME_S4A` (code) / `EM_WELCOME_S4B` (no code) | First-purchase incentive | Variant A: single-use 10%-off code, valid through midnight on the 7th day after send, date stated in copy. Variant B for `discount-abusers`: same layout, free shipping reminder and bestsellers, no code. |
| 5 | T+14d | Email | `EM_WELCOME_S5A` / `EM_WELCOME_S5B` | Expiry nudge, then exit | Variant A: "Your code expires tonight" with the code repeated (it lands on the last valid day). Variant B: new-arrivals edit, no urgency language. User exits after this step. |

**Per-step notes**

- **Splits:** Step 2 uses an Audience Path: push-enabled users get `PSH_WELCOME_S2`, everyone else gets `IAM_WELCOME_S2` queued for their next session. Step 4 uses a Decision Split on the `discount-abusers` segment; the split carries through to step 5 so the variants stay consistent.
- **Quiet hours:** Step 1 sends immediately, day or night: a welcome email that arrives eight hours late loses most of its open-rate advantage. Steps 2 through 5 land at 10 a.m. user local time. Canvas-level quiet hours apply to every step, so the build turns them off and times steps 2 through 5 with delay steps that release at a set local time instead (see `BUILD.md`).
- **Frequency caps:** The canvas counts toward global frequency caps. A brand-new subscriber rarely has cap history, so step 1 almost never gets capped. A user capped at a later step does not receive that step, which matters most at step 4: if your caps are tight, confirm that promotional campaigns are not crowding out the incentive for new subscribers.
- **Incentives:** 10% off, one use, from a Braze promotion code list (`northwind_welcome10`), valid through the end of the 7th day after send, excludes sale items and gift cards. Margin fence: the discount applies only to categories with gross margin at or above 35% after the discount; the commerce platform enforces this, not Liquid.

## Guardrails

- Email steps check `email_subscribe` is `subscribed` or `opted_in` (Braze does this by default for marketing email; do not override it).
- Push only to users who are push-enabled for the app; never nag for push permission more than once in this series.
- The code expiry date shown in the email must match the date the commerce platform enforces. Render it from the same send-time calculation, and never extend a code manually after launch.
- No "last chance" language unless something actually expires. Variant B never carries urgency copy.
- If the promotion code list is empty, step 4 aborts rather than sending an email with a blank code. Monitor list inventory (alert at 20% remaining).
- One discount per new customer: the code is single-use, and the commerce platform blocks stacking with other welcome or referral codes.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | 30-day first-purchase rate: share of entrants who make a first purchase within 30 days of entry. Starting target: +2 to +4 points of lift over control (a hypothesis to test, not a promise). |
| Secondary metrics | Revenue per entrant (net of discount), code redemption rate, share of first purchases made before step 4 (full-price conversions), 90-day unsubscribe rate of the cohort |
| Control design | 10% global control at canvas entry. Control users receive no welcome messages but still receive transactional email. |
| Attribution window | 30 days from canvas entry |
| Conversion event | Primary: `purchase`, deadline 30 days |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, step 1 hard-bounce rate <2% (a higher rate signals a list-quality problem at the capture source) |
| Kill criteria | Complaint rate above 0.1% on any send, or first-purchase rate at or below control after two full 30-day cycles |

Watch the full-price share. If most first purchases land after step 4, the incentive is doing the work and you should test moving it later or cutting it to free shipping. If most land before step 4, the code is mostly subsidy; test variant B for everyone.
