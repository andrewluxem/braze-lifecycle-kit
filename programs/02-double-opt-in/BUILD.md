# 02 · Double Opt-In: Build Recipe

Build time: about 45 minutes for both canvases. Build Canvas A first (you need its email steps to exist before Canvas B can reference their alias), but launch Canvas B first, so no click is ever missed.

## Before you start

- [ ] Signup forms set `email_subscribe` to `subscribed` (not `opted_in`) and fire custom event `email_subscribed` with `signup_source`. Confirm that no form, integration, or import writes `opted_in` directly.
- [ ] Custom attributes exist: `doi_status` (string), `doi_confirmed_at` (time). Custom event `email_confirmed` exists (it will be written by Canvas B).
- [ ] A confirmation landing page exists at `https://www.example.com/subscribe/confirmed` (plain thank-you, optional preference-center link). For the hardened variant, a page at `https://www.example.com/subscribe/confirm` with a Confirm button that fires `email_confirm_clicked`.
- [ ] Signup forms have a CAPTCHA or honeypot field.
- [ ] Content blocks exist: `CB_GLOBAL_HEADER_MINIMAL`, `CB_GLOBAL_FOOTER_MINIMAL` (logo, postal address, a line explaining why they received this email, and an unsubscribe link; no navigation or promotions).
- [ ] Decide with your privacy owner what "expired" means for your jurisdictions: leave the address `subscribed` but unmailable (default here), or set it to `unsubscribed`.
- [ ] Plan the switch: every promotional campaign and canvas moves to **Send to opted-in users only** on the day Canvas A launches. Make the list of campaigns to change before launch day.

## Canvas A: `CANVAS | Double Opt-In Request | v1`

1. **Create the canvas.** **Messaging > Canvas > Create Canvas**. Name `CANVAS | Double Opt-In Request | v1`. Tags `lifecycle`, `consent`, `always-on`.

2. **Conversion event.** Primary: **Performs Custom Event** `email_confirmed`, deadline 7 days.

3. **Entry schedule.** **Action-Based**, trigger **Perform Custom Event** `email_subscribed`. Always on. User local time zone.

4. **Target audience.** Filters (AND):
   - **Email Available** is `true`
   - **Email Subscription Status** is `subscribed`
   - Custom attribute `doi_status` is not `confirmed`
   - Segment is not `internal-test-users`

5. **Entry controls.** **Allow re-entry** ON, after **7 days**. Set an entry rate limit that exceeds your normal peak signups per minute by 2x to 3x; a burst beyond it is almost always a bot.

6. **Exit criteria.** Exception events: **Performs Custom Event** `email_confirmed`; **Email Subscription Status** changes to `unsubscribed`.

7. **Send settings.**
   - Subscription settings: **subscribed or opted in** (these users are `subscribed`; opted-in-only would block every confirmation email).
   - Quiet hours: OFF at canvas level. Step 2 is timed by its delay.
   - Frequency capping: turn it OFF for this canvas. A confirmation email must never lose a cap slot to a promotion.

8. **Step 1: Message.** Add **Message** step `EM_DOI_S1`, sent immediately. Paste block `EM_DOI_S1` from [`snippets.liquid`](snippets.liquid). In the email editor's **Link Management** tab, set the alias for the Confirm button link to exactly `confirm-subscription`. Turn link tracking on. No control group before this step.

9. **Delay.** Add **Delay** step: 2 days, release at 10:00 user local time.

10. **Decision Split.** Add **Decision Split** named `SPLIT_DOI_PENDING_48H`: custom attribute `doi_status` is not `confirmed` AND **Email Subscription Status** is `subscribed`. Yes = still pending. No = end (they confirmed and should already have exited).

11. **Experiment Paths (control).** On the Yes path, add **Experiment Paths** with two paths: Path 1 at 90%, **Control** at 10%. Winning-path selection OFF. This is the program's 10% control, applied to the reminder.

12. **Step 2: Message.** On Path 1, add **Message** step `EM_DOI_S2`. Paste block `EM_DOI_S2`. Set the Confirm link alias to `confirm-subscription` again (aliases are per message).

13. **Delay to expiry.** Rejoin the Path 1 and Control paths into one **Delay** step: 5 days (T+7d total).

14. **Decision Split.** Add **Decision Split** named `SPLIT_DOI_PENDING_7D`, same filters as step 10.

15. **Step 3: User Update (expire).** On the Yes path, add **User Update** step `UPD_DOI_EXPIRE`, Advanced JSON editor, paste block `UPD_DOI_EXPIRE`:
    ```json
    {"attributes": [{"doi_status": "expired"}]}
    ```
    If your policy is to unsubscribe expired addresses, add `"email_subscribe": "unsubscribed"` to the same object.

16. **End.** Both split paths end.

## Canvas B: `CANVAS | Double Opt-In Confirm | v1`

17. **Create the canvas.** Name `CANVAS | Double Opt-In Confirm | v1`. Tags `lifecycle`, `consent`, `always-on`.

18. **Entry schedule.** **Action-Based**, trigger **Interact with Campaign** (for Canvas steps, the interaction option for canvas messages): select `CANVAS | Double Opt-In Request | v1`, message steps `EM_DOI_S1` and `EM_DOI_S2`, interaction **Clicked Alias**, alias `confirm-subscription`. If your workspace does not list alias clicks for canvas steps as an entry trigger, use the hardened variant: trigger **Perform Custom Event** `email_confirm_clicked`, and swap the Confirm link in both emails for block `FRAGMENT_DOI_CONFIRM_LINK`.

19. **Target audience.** **Email Subscription Status** is not `opted_in`. Segment is not `internal-test-users` is optional here; test users need to confirm too.

20. **Entry controls.** Re-entry OFF.

21. **No control group.** This canvas records consent. Set Control to 0%.

22. **Step 1: User Update.** Add **User Update** step `UPD_DOI_CONFIRM`, Advanced JSON editor. The required payload is:
    ```json
    {"attributes": [{"email_subscribe": "opted_in"}]}
    ```
    Use the full block `UPD_DOI_CONFIRM` from the snippets file, which also stamps `doi_status`, `doi_confirmed_at`, and logs the `email_confirmed` event that exits Canvas A and enters program 01.

23. **End.** No messages. The landing page is the user's confirmation.

## Launch sequence

24. Launch Canvas B.
25. Launch Canvas A.
26. Switch every promotional campaign and canvas to **opted-in users only** (from your list in Before you start).
27. Change program 01's trigger from `email_subscribed` to `email_confirmed`.

## Program-specific QA additions

Run [`library/qa-checklist.md`](../../library/qa-checklist.md) first, then these:

- [ ] Subscribe a fresh test address through each signup form. `EM_DOI_S1` arrives within 2 minutes at any hour.
- [ ] Clicking Confirm in `EM_DOI_S1` sets `email_subscribe` to `opted_in`, `doi_status` to `confirmed`, `doi_confirmed_at` to the click time, logs `email_confirmed`, removes the user from Canvas A, and enters program 01.
- [ ] Same test from `EM_DOI_S2`. The alias is set on both messages (check each message's Link Management tab, not just one).
- [ ] Clicking Confirm twice does not re-enter program 01 or overwrite `doi_confirmed_at`.
- [ ] A user who never clicks receives `EM_DOI_S2` at 48 hours (or nothing, if in the control path) and has `doi_status` = `expired` at 7 days.
- [ ] An expired user who re-subscribes on day 8 re-enters Canvas A; a user who re-subscribes on day 3 does not get a second `EM_DOI_S1`.
- [ ] Submit the same address through a form 10 times in a minute. At most one confirmation email sends.
- [ ] A test promotional campaign set to **opted-in only** does not reach a pending or expired user.
- [ ] Confirmation email contains no product, price, or promotion content.
- [ ] Hardened variant only: send `EM_DOI_S1` to an address behind a link-scanning mail gateway. Confirmation happens only when the Confirm button on the page is pressed, not when the email arrives.
