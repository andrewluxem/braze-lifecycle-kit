# 01 · Welcome / Onboarding: Build Recipe

Build time: about 60 minutes. Build in a staging or sandbox workspace first if you have one.

## Before you start

- [ ] Custom events `signed_up` and `email_subscribed` are firing in production with `signup_source` populated. Check in **Data Settings > Custom Events** and in a test user's profile.
- [ ] Segments exist: `discount-abusers`, `internal-test-users` (see [`library/segment-recipes.md`](../../library/segment-recipes.md)).
- [ ] Promotion code list `northwind_welcome10` is uploaded with single-use codes that the commerce platform expires at 23:59 on the 7th day after issue (store time zone, or later, so a code never dies before the date the email states). Low-inventory alert set at 20% remaining.
- [ ] Content blocks exist: `CB_GLOBAL_HEADER`, `CB_GLOBAL_FOOTER`, `CB_GLOBAL_NEW_ARRIVALS`, `CB_WELCOME_BESTSELLERS` (merchandiser-updated weekly), `CB_WELCOME_BESTSELLERS_FALLBACK` (static, never edited to empty).
- [ ] A preference center exists (referenced here as `northwind_preferences`).
- [ ] The commerce platform sends custom event `promo_code_redeemed` (properties `code_list`, `order_id`) and sets boolean custom attribute `welcome_code_redeemed` = `true` when a `northwind_welcome10` code is used. Step 5A depends on it.

## Canvas construction

1. **Create the canvas.** Go to **Messaging > Canvas > Create Canvas**. Name it `CANVAS | Welcome Series | v1`. Add tags `lifecycle`, `welcome`, `always-on`.

2. **Set conversion events.** In **Basics**, add primary conversion event **Makes Purchase** (any product) with a 30-day deadline. Add secondary conversion event **Performs Custom Event** `promo_code_redeemed` with `code_list` = `northwind_welcome10`, 30-day deadline, so you can separate code-driven from full-price conversions.

3. **Set entry schedule.** Choose **Action-Based**. Add two trigger actions:
   - **Perform Custom Event** `signed_up`
   - **Perform Custom Event** `email_subscribed`

   If program 02 is live, replace `email_subscribed` with `email_confirmed`. Entry window: start at launch, no end date. Time zone: user local time.

4. **Set the target audience.** Add filters, joined with AND:
   - **Email Available** is `true`
   - **Email Subscription Status** is not `unsubscribed`
   - **Number of Purchases** equals `0` (lifetime)
   - Segment **is not** `internal-test-users`

   Do not exclude `discount-abusers` here; they route at step 4.

5. **Set entry controls.** Turn **Allow users to re-enter this Canvas** OFF. This canvas is once per lifetime. Leave the entry limit unlimited, and set a daily entry cap only if a bulk list import is planned (a list import that fires `email_subscribed` for 50,000 users at once will flood step 1).

6. **Set exit criteria.** Add exception events: **Makes Purchase** (any product), and **Email Subscription Status** changes to `unsubscribed`. Users exit from any step.

7. **Set send settings.**
   - Subscription settings: send to users who are **subscribed or opted in**.
   - Rate limiting: none for normal traffic. If you expect a signup spike (a giveaway or a large import), set a limit that your support team and site can absorb, such as 10,000 per minute.
   - Quiet hours: **OFF** at the canvas level. Step 1 must send immediately; later steps are timed by the delay steps below.
   - Frequency capping: leave the canvas subject to global caps.

8. **Add the control group.** In the canvas builder, keep one variant (`Variant 1`, 90%) and set **Control** to 10%. Control users enter, are tracked for conversion, and receive nothing from this canvas.

9. **Step 1: Message.** Add a **Message** step named `EM_WELCOME_S1`. Channel: Email. Paste block `EM_WELCOME_S1` from [`snippets.liquid`](snippets.liquid). Subject from the snippet, preheader set. Link aliasing on. Send immediately (no delay before this step).

10. **Delay 1.** Add a **Delay** step: 1 day, then release at **10:00 in the user's local time**.

11. **Audience Paths for step 2.** Add an **Audience Paths** step with two groups:
    - `push-enabled`: filter **Foreground Push Enabled** is `true` (or use segment `push-enabled`).
    - Everyone else.

12. **Step 2a: Message.** On the push path, add a **Message** step `PSH_WELCOME_S2` (iOS and Android push). Paste block `PSH_WELCOME_S2`. Deep link to the category URL from the snippet.

13. **Step 2b: Message.** On the everyone-else path, add a **Message** step `IAM_WELCOME_S2` (in-app message, slideup). Trigger: **Session start**. Set message expiration to 3 days so a user who does not open the app never sees a stale value-prop card. Paste block `IAM_WELCOME_S2`. Both paths reconnect into the next delay.

14. **Delay 2.** Add a **Delay** step: 2 days (T+3d total), release at 10:00 user local time.

15. **Step 3: Message.** Add **Message** step `EM_WELCOME_S3`. Paste block `EM_WELCOME_S3` (social proof, three bestsellers with ratings).

16. **Delay 3.** Add a **Delay** step: 4 days (T+7d), release at 10:00 user local time.

17. **Decision split for the incentive.** Add a **Decision Split** step: **Segment** `discount-abusers`. Yes path = no-code branch. No path = code branch.

18. **Step 4A: User update (assign code).** On the No path, add a **User Update** step named `UPD_WELCOME_S4A`. Advanced JSON editor, paste block `UPD_WELCOME_S4A` from the snippets file:
    ```json
    {"attributes": [{"welcome_code": "{% promotion('northwind_welcome10') %}", "welcome_code_sent_at": "{{ 'now' | date: '%Y-%m-%dT%H:%M:%SZ' }}"}]}
    ```
    This pulls one single-use code and stamps the send time on the profile, so step 4A and step 5A show the same code and the same true expiry, and support can verify a customer's expiry claim. If your workspace shows stale attributes in the step right after a User Update, add a 1-minute delay here.

19. **Step 4A: Message (code).** Add **Message** step `EM_WELCOME_S4A`. Paste block `EM_WELCOME_S4A`. It reads `welcome_code` and `welcome_code_sent_at`, aborts if either is blank (empty code list or failed update), and renders the expiry as sent-at plus 7 days.

20. **Step 4B: Message (no code).** On the Yes path, add **Message** step `EM_WELCOME_S4B`. Paste block `EM_WELCOME_S4B`.

21. **Delay 4.** On each path, add a **Delay** step: 7 days (T+14d), release at 10:00 user local time. Keep the paths separate so each variant gets its matching step 5.

22. **Step 5A: Message.** On the code path, add **Message** step `EM_WELCOME_S5A` (expiry nudge). Paste block `EM_WELCOME_S5A`. It aborts if the user has already redeemed the code (checks `welcome_code_redeemed`).

23. **Step 5B: Message.** On the no-code path, add **Message** step `EM_WELCOME_S5B` (new arrivals). Paste block `EM_WELCOME_S5B`.

24. **End.** Users leave the canvas after step 5. No further steps.

25. **Re-eligibility check.** Confirm in the **Summary** view: re-entry off, control 10%, exit events present, quiet hours off, conversion events set.

26. **Seed and launch.** Add seed group `seeds-lifecycle` to email steps. Save as draft, run the QA below, then launch.

## Program-specific QA additions

Run [`library/qa-checklist.md`](../../library/qa-checklist.md) first, then these:

- [ ] Fire `signed_up` for a fresh test user with `signup_source` = `checkout`, `popup`, and missing. Step 1 renders the right intro line for each and a neutral fallback when the property is absent.
- [ ] Fire both `signed_up` and `email_subscribed` for the same test user within one minute. The user enters once, not twice.
- [ ] A test user with one prior purchase is blocked at entry.
- [ ] A test user who purchases after step 1 exits and receives no further welcome messages. Confirm in the user's Canvas history.
- [ ] A test user in `discount-abusers` receives `EM_WELCOME_S4B` and `EM_WELCOME_S5B`, never a code.
- [ ] `UPD_WELCOME_S4A` writes a real code to the profile, the code works once at checkout, fails a second time, and fails on excluded categories.
- [ ] The expiry date in step 4A equals send date plus 7 days in the user's time zone, and the commerce platform accepts the code until 23:59 on that date.
- [ ] Step 5A lands on the code's last valid day and says "tonight"; if the user was capped and step 5A runs a day late, it aborts.
- [ ] Temporarily point `UPD_WELCOME_S4A` at an empty test code list: `welcome_code` stays blank, `EM_WELCOME_S4A` aborts with its logged reason, and no email sends.
- [ ] Step 4A and step 5A show the identical code for the same test user.
- [ ] Step 5A does not send to a user who already redeemed the code.
- [ ] A non-push-enabled test user receives the in-app message on next session and never receives the push.
- [ ] Signup at 11 p.m. local time: step 1 arrives immediately; step 2 arrives at 10 a.m. local time, never overnight.
- [ ] Preference center link in step 1 opens the correct preference center and saves changes back to the profile.
