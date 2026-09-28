# Pre-Launch QA Checklist

Run this for every canvas or campaign before launch, then run the program-specific additions at the bottom of its `BUILD.md`. Copy the checklist into the launch ticket and tick it there; an unticked box is a launch blocker unless the launch owner signs off on it in writing.

Previews are not QA. Liquid preview uses test data you typed, not the data your systems will send. Every item marked **(live)** requires a real event through a test user.

## Audience and entry

- [ ] **(live)** A real trigger event, sent from the production integration (SDK, backend, or `/users/track`), enters a test user into the canvas. Not a manually typed test event from the dashboard.
- [ ] The trigger event's properties arrive with the expected names, types, and nesting. Checked on the test user's profile.
- [ ] Entry audience filters are correct: at least one test user who should enter does, and at least one who should not (wrong subscription state, excluded segment) does not.
- [ ] Every exclusion segment exists, has a plausible size, and is spelled exactly as referenced.
- [ ] `internal-test-users` is excluded.
- [ ] Re-eligibility matches `PROGRAM.md` (never / N days). Tested by firing the trigger twice inside the window.
- [ ] Every exit criterion works: **(live)** fire each exit event for a test user mid-canvas and confirm they leave and receive nothing further.
- [ ] Entry rate limit or daily cap is set if a bulk import or spike could flood the canvas.
- [ ] Canvas priority versus related programs is documented (for example, cart abandonment above browse abandonment), and the mechanism that enforces it is tested.

## Messages

- [ ] Every link resolves (no 404s, no staging URLs, no `example.com` left in a real build), carries the right UTM parameters, and dynamic values in URLs are `url_encode`d.
- [ ] Every Liquid output has a fallback. Tested by previewing a user with blank first name, blank custom attributes, and missing event properties.
- [ ] Every block that cannot render its core content aborts with a named reason. **(live)** Forced at least one abort and found it in the message activity log.
- [ ] Images load, have alt text, and have width set. The email reads sensibly with images off.
- [ ] Subject, preheader, and from name are set for every email, and none of them renders a raw Liquid tag or an empty merge.
- [ ] Rendering checked on the major clients your audience uses (desktop and mobile), plus dark mode.
- [ ] Push: title and body within length limits for iOS and Android, deep links open the right screen on both platforms, fallback URL set.
- [ ] SMS: under 160 GSM characters (or intended multi-part), link shortening on, opt-out language included.
- [ ] In-app: trigger, expiry, and display priority set; message cannot show after the reason for it has passed.
- [ ] Quiet hours set as specified, and **(live)** a trigger sent inside quiet hours delivers at the expected local time.
- [ ] Frequency capping behavior is deliberate: on or off per `PROGRAM.md`, and the team knows what happens to a capped step.
- [ ] Every message and step is named per [`naming-taxonomy.md`](naming-taxonomy.md).
- [ ] Seed group added to email steps.

## Incentives and dynamic content

- [ ] Promotion codes are single-use, the list has enough inventory for at least 2x the expected monthly volume, and a low-inventory alert is set.
- [ ] **(live)** A pulled code works once at checkout, fails the second time, and respects exclusions (sale items, categories under the margin floor).
- [ ] The expiry shown in the message equals the expiry the commerce platform enforces, in the user's time zone.
- [ ] `discount-abusers` route to the no-incentive variant. **(live)** Tested with a test user in the segment.
- [ ] Margin fence is enforced by the commerce platform or a live check, not only by copy.
- [ ] Connected Content failure mode tested: endpoint returning 500, timing out, and returning 200 with an empty payload. Each produces the documented fallback or abort.
- [ ] Connected Content uses stored credentials; no key or token appears in any message body.
- [ ] Canvas rate limit keeps Connected Content calls under the endpoint's capacity.
- [ ] Price staleness handled: prices come from the catalog or a live API at send time, or are not shown.
- [ ] Catalog-driven content handles a product missing from the catalog and an out-of-stock product.

## Compliance

These are operational checks, not legal advice. Your counsel owns the rules for each market you send to.

- [ ] Every promotional email checks subscription status (opted-in only, where program 02 is live).
- [ ] SMS sends only to the SMS marketing subscription group, and consent capture records when and where consent was given.
- [ ] Every promotional email has a working one-click unsubscribe link and a preference center link. **(live)** Clicked unsubscribe on a test user and confirmed the status changed.
- [ ] List-unsubscribe headers are enabled for promotional email.
- [ ] Physical mailing address present in promotional email footers.
- [ ] Transactional messages are classified as transactional, contain no promotional content, and are sent on the transactional path.
- [ ] No sensitive-category data (health, finances, anything a shared inbox should not reveal) appears in subject lines or push text.
- [ ] Data used for personalization was collected for that purpose.

## Measurement

- [ ] Control group set at 10% (or the documented exception for transactional or consent messages).
- [ ] Primary conversion event and deadline match `PROGRAM.md`.
- [ ] UTM and campaign naming allow revenue attribution in your analytics tool.
- [ ] Guardrail thresholds (unsubscribe under 0.3% per send, complaint under 0.1% per send) have alerts or a named person checking them for the first two weeks.
- [ ] Baseline for the primary KPI recorded before launch.
- [ ] Reporting cut agreed: which dashboard, which cadence, who reads it. See [`measurement-framework.md`](measurement-framework.md).

## Launch

- [ ] Canvas saved as draft and reviewed by a second person who did not build it.
- [ ] Owner named in the canvas description, with a backup.
- [ ] Kill criteria written in the canvas description ("pause if complaint rate exceeds 0.1% on any send").
- [ ] Launch time chosen so the owner is available for the first 24 hours of sends. Not Friday afternoon.
- [ ] Downstream teams told: customer service knows what customers will receive and what codes exist.
- [ ] Activated. First sends spot-checked in the seed inbox and the message activity log within the first hour.
- [ ] Day 3 and day 14 check-ins scheduled.
