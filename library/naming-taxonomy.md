# Naming Taxonomy

Names are the only documentation most Braze workspaces have. A good name tells you what a thing is, which program owns it, and where it sits in the sequence, from a list view, a report export, or a Currents row, without opening it. Every program in this kit uses the rules below.

General rules:

- One name, one thing. Never reuse a name for a different object, even after the old one is archived.
- Names describe the object, not the campaign theme. "Spring" belongs in a tag, not a canvas name.
- Change the version number, not the name, when a rebuild changes logic. Archive the old version; do not edit a live canvas's structure beyond what Braze allows safely.
- ASCII only. No emoji, no smart quotes, no trailing spaces. Names end up in CSV exports and API calls.

## Canvases

**Pattern:** `CANVAS | <Program> | v<N>`

- `<Program>`: Title Case, plain words, matches the program folder's name.
- `v<N>`: integer version, bumped on any structural change (new step, new split, changed entry or exit). Copy edits do not bump the version.

| Good | Bad | Why bad |
|---|---|---|
| `CANVAS \| Welcome Series \| v1` | `Welcome Series NEW` | No version, "NEW" ages badly |
| `CANVAS \| Cart Abandonment \| v2` | `CANVAS \| Cart Abandonment \| v2 FINAL (use this one)` | Status belongs in tags and archive state |
| `CANVAS \| Double Opt-In Request \| v1` | `CANVAS \| DOI A \| v1` | Abbreviations nobody outside the team can read |

## Campaigns and message steps

**Pattern:** `<CHAN>_<PROGRAM>_<STEP><VARIANT>`

| Part | Values | Notes |
|---|---|---|
| `CHAN` | `EM` email, `PSH` push, `SMS` SMS/MMS, `IAM` in-app message, `TRX` transactional (any channel), `WH` webhook | `TRX` replaces the channel code for transactional messages so they are easy to find and exclude from promo reporting |
| `PROGRAM` | Uppercase short program key: `WELCOME`, `DOI`, `BROWSE`, `CART`, `POSTPURCH`, `WINBACK`, `CHURN`, `SUNSET`, `BIS`, `PRICEDROP`, `REFERRAL`, `LOYALTY_TIER`, `LOYALTY_EXPIRY`, `MILESTONE`, `REPERM` | Keep it to one or two words joined by `_` |
| `STEP` | `S1`, `S2`, ... in send order | Parallel channels in the same step share the step number |
| `VARIANT` | Optional `A`, `B`, ... | Use for split branches (code vs no-code) or experiment arms |

Examples: `EM_WELCOME_S1`, `PSH_WELCOME_S2`, `IAM_WELCOME_S2`, `EM_CART_S3A`, `EM_CART_S3B`, `SMS_CART_S1`, `WH_POSTPURCH_S2`, `TRX_ORDER_CONFIRMATION` (transactional messages that stand alone may use a descriptive suffix instead of a step).

Name the Message step in the canvas and the message inside it identically. If a step holds two channels (email and push), give the step the email name and each message its own channel name.

### Non-message canvas steps

Braze does not require names on delays, splits, or user updates, but naming the ones that carry logic makes the canvas readable and the audit trail clear.

| Step type | Prefix | Example |
|---|---|---|
| User Update | `UPD_` | `UPD_WELCOME_S4A`, `UPD_DOI_CONFIRM` |
| Decision Split / Audience Paths | `SPLIT_` | `SPLIT_CART_INCENTIVE`, `SPLIT_DOI_PENDING_48H` |
| Action Paths | `ACT_` | `ACT_POSTPURCH_DELIVERY` |
| Delay (only when it carries logic) | `DLY_` | `DLY_CART_QUALIFY`, `DLY_BROWSE_QUALIFY` |
| Experiment Paths | `EXP_` | `EXP_WELCOME_S4_TIMING` |

Snippet fragments that are pasted into another message and never sent alone use `FRAGMENT_<PROGRAM>_<DESCRIPTOR>` (for example `FRAGMENT_DOI_CONFIRM_LINK`).

## Segments

**Pattern:** lowercase, hyphenated, behavior first, window last.

- Lead with what the user did or is: `lapsed`, `purchased`, `email-engaged`, `discount-abusers`.
- End with the window when there is one: `-30d`, `-90-180d`.
- No program names in segment names. Segments are shared; a segment named after one program gets copied instead of reused.

| Good | Bad |
|---|---|
| `lapsed-90-180d` | `Winback Audience` |
| `email-engaged-90d` | `engaged` (engaged how, since when?) |
| `discount-abusers` | `Coupon Hunters - DO NOT SEND` |
| `high-intent-no-purchase-14d` | `HighIntent_NoPurch_14` |

Full definitions: [`segment-recipes.md`](segment-recipes.md).

## Content blocks

**Pattern:** `CB_<SCOPE>_<DESCRIPTOR>`

- `SCOPE`: `GLOBAL` for anything used across programs, otherwise the program key (`WELCOME`, `CART`, `POSTPURCH`), or `TRX` for transactional-only blocks.
- `DESCRIPTOR`: uppercase words joined with `_`.

Examples: `CB_GLOBAL_HEADER`, `CB_GLOBAL_FOOTER`, `CB_GLOBAL_HEADER_MINIMAL`, `CB_GLOBAL_FOOTER_MINIMAL`, `CB_GLOBAL_NEW_ARRIVALS`, `CB_TRX_HEADER`, `CB_WELCOME_BESTSELLERS_FALLBACK`, `CB_CART_SERVICE_PROMISES`, `CB_POSTPURCH_CROSSSELL_FALLBACK`.

Fallback blocks end in `_FALLBACK` so nobody deletes them for being "unused": they only render when something else fails.

## Tags

Lowercase, hyphenated, one concept per tag. Apply to every canvas and campaign.

| Tag group | Values |
|---|---|
| Lifecycle stage | `lifecycle`, `acquisition`, `retention`, `reactivation` |
| Program type | `welcome`, `consent`, `abandonment`, `post-purchase`, `winback`, `sunset`, `loyalty` |
| Cadence | `always-on`, `scheduled`, `one-off` |
| Message class | `transactional`, `promotional` |
| Business goal | `revenue`, `deliverability`, `sentiment` |

Tags are how you filter reports, so be consistent: `always-on` everywhere, never `always on` or `evergreen` in some places.

## Custom events

**Pattern:** `snake_case`, object then past-tense verb where natural, or verb then object when that is the established term.

Examples used in this kit: `signed_up`, `email_subscribed`, `email_confirmed`, `email_confirm_clicked`, `product_viewed`, `added_to_cart`, `cart_cleared`, `delivery_confirmed`, `return_initiated`, `review_submitted`, `promo_code_redeemed`, `subscription_cancel_initiated`, `subscription_saved`, `back_in_stock_signup`, `back_in_stock`, `price_dropped`, `referral_sent`, `loyalty_tier_upgraded`, `preferences_updated`.

Rules:

- Past tense for things that happened (`added_to_cart`, not `add_to_cart`).
- No channel or program in the event name. Events describe customer behavior; programs react to it.
- Property names are `snake_case` too (`cart_id`, `cart_total`, `product_id`, `signup_source`).
- Do not send PII (email, phone, address) as event properties. Use IDs.
- Purchases use Braze's native purchase object, not a custom `purchase` event, so revenue reporting works.

## Custom attributes

`snake_case`, noun first, with a type-revealing suffix for times and flags: `doi_status`, `doi_confirmed_at`, `welcome_code`, `welcome_code_sent_at`, `has_open_return`, `last_order_at`, `promo_suppressed`, `pref_topic_guides`, `pref_frequency`. Time attributes end in `_at`; booleans start with `has_`, `is_`, or read as a state (`promo_suppressed`).

## Promotion code lists and link aliases

- Promotion code lists: `<brand>_<program><value>`, lowercase: `northwind_welcome10`, `northwind_cart10`, `northwind_winback10`.
- Link aliases that drive logic (entry triggers, exits, segments): lowercase hyphenated verbs, stable across versions: `confirm-subscription`, `sunset-stay`.
