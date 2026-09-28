# Deliverability Checklist

Every program in this kit assumes your email reaches the inbox. This checklist covers the setup and habits that make that true. Run the authentication and infrastructure sections once (and after any DNS or ESP change), the warming section before any new IP, domain, or major volume increase, and the monitoring section continuously.

Mailbox provider requirements change. The largest mailbox providers raised bulk-sender requirements in 2024 (authentication, one-click unsubscribe, spam-rate limits), and others followed. Check each provider's current sender guidelines before a warming plan or a major change.

## Authentication

- [ ] **SPF** record on the sending domain includes Braze's sending infrastructure (per your Braze onboarding), with no more than 10 DNS lookups in total. Check with an SPF validator after every change.
- [ ] **DKIM** signing enabled with a 2048-bit key, aligned to the From domain (the `d=` domain matches or is a subdomain of the From domain).
- [ ] **DMARC** record published on the organizational domain with **`p=quarantine` minimum** (`p=reject` once reports are clean). `p=none` is monitoring, not protection.
  - Example: `v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc-reports@example.com; adkim=r; aspf=r`
- [ ] DMARC aggregate reports (`rua`) are received and reviewed at least monthly. Unknown senders using your domain get either authorized or blocked.
- [ ] **BIMI** considered once DMARC is at enforcement (optional; needs `p=quarantine` or `p=reject`).
- [ ] Reverse DNS (PTR) on dedicated IPs resolves to the sending subdomain.

## Infrastructure

- [ ] **Dedicated sending subdomain** for marketing (Northwind: `news.example.com`), separate from the corporate domain that sends employee mail. A reputation problem in marketing must not take down invoices and password resets.
- [ ] Transactional mail on its own subdomain (`orders.example.com`) and, at volume, its own IP pool, so a promotional complaint spike never delays an order confirmation.
- [ ] Click-tracking domain branded and on the same organizational domain (`click.news.example.com`), with HTTPS.
- [ ] From name and From address stable and recognizable. Reply-to monitored, or at minimum auto-replies with a help link. No `noreply@` on marketing mail.
- [ ] **One-click unsubscribe:** `List-Unsubscribe` and `List-Unsubscribe-Post` headers enabled on all promotional mail, and unsubscribes processed immediately.
- [ ] Hard bounces suppressed automatically after the first bounce; repeated soft bounces (5 or more over 30 days) suppressed.

## IP and domain warming

For a new dedicated IP, a new sending domain, or a volume increase of more than about 2x.

- [ ] **Plan for 4 to 6 weeks.** Faster warming works only for senders with an excellent existing domain reputation, and it is not worth the risk.
- [ ] **Start with the most engaged subscribers.** Week 1 goes only to `email-engaged-90d` recipients who clicked in the last 30 days. Widen the engagement window each week; mail no one in `unengaged-180d` during warming, ever.
- [ ] Daily volume roughly doubles every 2 to 3 days while metrics hold. Example ramp (adjust to your list size): 2,000 on day 1, 5,000 on day 3, 10,000 on day 5, 25,000 on day 8, 50,000 on day 11, then 50% increases until you reach full volume.
- [ ] Volume spread across mailbox providers in proportion to your list. Watch each provider separately; one provider throttling is a signal to hold that provider's volume, not the whole ramp.
- [ ] **Hold or step back** when any provider shows: deferrals or throttling, complaint rate above 0.1%, bounce rate above 2%, or a drop in open rate of more than a third versus the prior step.
- [ ] Warm with your best content (welcome, post-purchase, top newsletters), not a clearance blast.
- [ ] Program 08 (sunset) ran before warming began, so the list you warm with is a list worth keeping.

## Ongoing list hygiene

- [ ] **Sunset program running** (program 08): unengaged addresses are re-permissioned and then suppressed from promotional mail on a weekly schedule.
- [ ] **Engagement-filtered launches:** every new program or large one-off send goes first to engaged segments, then widens after 24 to 48 hours of clean metrics.
- [ ] Double opt-in (program 02) for new subscribers, or at minimum real-time address validation and bot protection on signup forms.
- [ ] No purchased, rented, appended, or scraped lists. Ever.
- [ ] Role addresses (`info@`, `sales@`) and known disposable domains flagged at capture.

## Monitoring and alert thresholds

- [ ] **Complaint rate:** alert above **0.1% per send**; treat anything approaching 0.3% as an incident (some providers begin filtering at that level). Measure against delivered mail, per provider where possible.
- [ ] **Unsubscribe rate:** alert above **0.3% per send**. A spike usually means wrong audience, wrong frequency, or broken expectations, not bad copy.
- [ ] **Bounce rate:** alert above 2% hard bounces on any send (list-quality problem at the source).
- [ ] **Inbox placement by mailbox provider:** measured with seed tests or panel data at least weekly, and reported per provider. An average hides the provider that is filtering you.
- [ ] Each major mailbox provider's postmaster or sender-reputation dashboard connected and checked weekly for domain and IP reputation and spam rate.
- [ ] Blocklist monitoring on sending IPs and domains, with an owner who acts on a listing the same day.
- [ ] Every alert has a named owner and a written first response: pause the offending send, identify the segment, fix the cause, re-launch to engaged users first.

## Before every large send

- [ ] Audience excludes `unengaged-180d` and `promo_suppressed` users.
- [ ] Volume is within 2x of your recent daily average, or you have a ramp.
- [ ] Authentication passes on a seed message (check headers: `spf=pass`, `dkim=pass`, `dmarc=pass`).
- [ ] Unsubscribe link and header work on the final build.
