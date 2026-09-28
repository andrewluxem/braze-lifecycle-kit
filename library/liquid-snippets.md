# Liquid Snippets

Drop-in Braze Liquid patterns used across the kit. Every pattern follows three rules:

1. **Every output has a fallback.** A blank variable renders a sensible default, never an empty gap or a raw tag.
2. **Every external call has a failure path.** Connected Content checks its status code and falls back to static content, or aborts when no honest fallback exists.
3. **Abort beats broken.** If a message cannot say what it is supposed to say (no cart, no code, no order number), `abort_message` stops that send and logs why. A skipped message costs nothing; a broken one costs trust.

Braze syntax reminders: attributes use `${...}` inside the braces. Standard attributes: `{{${first_name}}}`. Custom attributes: `{{custom_attribute.${loyalty_tier}}}`. Content blocks: `{{content_blocks.${CB_GLOBAL_HEADER}}}`.

## 1. Personalization with fallback

```liquid
{%- assign fname = ${first_name} | strip | default: 'there' -%}
<h1>Hi {{fname}},</h1>
```

`strip` runs first so a first name of `" "` becomes blank and the default applies. Do not add `capitalize`: it can lowercase the rest of the string, which turns "McKenna" into "Mckenna". Fix name casing at the source.

Custom attribute with fallback:

```liquid
{%- assign tier = custom_attribute.${loyalty_tier} | strip | default: 'Member' -%}
```

In a subject line, where there is no room for an `assign`, inline it and pick a default that reads naturally without the name:

```liquid
{{${first_name} | strip | default: 'Hey'}}, your order shipped
```

Conditional greeting, when a default word would read awkwardly:

```liquid
Welcome to Northwind{% if ${first_name} != blank %}, {{${first_name} | strip}}{% endif %}.
```

## 2. Safe event-property access

Where the triggering data lives depends on how the message was sent:

| Send type | Syntax | Available in |
|---|---|---|
| Canvas, action-based entry | `{{canvas_entry_properties.${name}}}` | Every step of the canvas |
| Canvas, Action Paths step | `{{event_properties.${name}}}` | Only the first Message step after the Action Path that the event triggered |
| Action-based campaign | `{{event_properties.${name}}}` | That campaign's message |
| API-triggered campaign or canvas | `{{api_trigger_properties.${name}}}` / `{{canvas_entry_properties.${name}}}` | That message / every canvas step |
| Scheduled campaign or canvas | None | Use profile attributes only |

Newer Braze workspaces may also expose entry data as canvas context variables. The fallback discipline below applies either way.

Pattern: read once, strip, default or abort.

```liquid
{%- assign pid = canvas_entry_properties.${product_id} | strip -%}
{%- if pid == blank -%}
  {% abort_message('EM_BROWSE_S1: product_id missing') %}
{%- endif -%}
{%- assign product_name = canvas_entry_properties.${name} | strip | default: 'the item you viewed' -%}
```

Nested properties (arrays of objects) work with normal Liquid access:

```liquid
{%- assign cart_items = canvas_entry_properties.${items} -%}
{%- if cart_items == blank or cart_items.size == 0 -%}
  {% abort_message('no cart items') %}
{%- endif -%}
{%- assign lead_name = cart_items[0].name | strip | default: 'your item' -%}
```

Decision Splits and Audience Paths filter on profile data, not entry properties. If a split depends on an entry property, copy it onto the profile with a User Update step first (see `UPD_POSTPURCH_FLAGS` in program 05).

## 3. Connected Content with error handling

```liquid
{% connected_content https://api.example.com/v1/carts/{{cart_id | url_encode}} :method get :basic_auth northwind_cart_api :content_type application/json :retry :save live_cart %}
{%- if live_cart.__http_status_code__ == 200 and live_cart.items != nil -%}
  {%- assign cart_items = live_cart.items -%}
{%- else -%}
  {%- comment -%} Static fallback: entry snapshot. {%- endcomment -%}
  {%- assign cart_items = canvas_entry_properties.${items} -%}
{%- endif -%}
{%- if cart_items == blank or cart_items.size == 0 -%}
  {% abort_message('cart API failed and no fallback items') %}
{%- endif -%}
```

Rules:

- **Credentials:** store API keys under **Settings > Connected Content** and reference them with `:basic_auth <credential_name>` (or a stored token header). Never paste a key into Liquid; message bodies are visible to everyone with dashboard access and end up in exports.
- **Status check:** check `__http_status_code__` and the presence of the field you need. A 200 with an error body is still a failure.
- **`:retry`** retries on server errors. Pair it with a rate limit on the canvas so retries do not amplify an outage.
- **`:cache_max_age <seconds>`** caches GET responses by URL. Use it for data shared across users (a bestsellers list: cache 15 minutes). For per-user data the URL is unique per user, so caching mainly helps retries; keep it short or omit it for data that must be fresh (live cart, inventory).
- **Fallback choice:** decide per call. Recommendations fail to a static bestseller block. A live cart fails to the entry snapshot. A price or stock lookup fails to hiding the price, never to a guessed one.
- **Capacity:** every recipient makes a call at send time. Set the canvas rate limit below what the endpoint can serve.

Static fallback for a recommendations module:

```liquid
{% connected_content https://api.example.com/v1/recommendations?user={{${user_id} | url_encode}}&limit=3 :basic_auth northwind_recs_api :cache_max_age 900 :retry :save recs %}
{%- if recs.__http_status_code__ == 200 and recs.products.size > 0 -%}
  {%- for p in recs.products limit: 3 -%}
    <a href="{{p.url}}">{{p.name | strip | default: 'Recommended for you' | escape}}</a>
  {%- endfor -%}
{%- else -%}
  {{content_blocks.${CB_GLOBAL_BESTSELLERS_FALLBACK}}}
{%- endif -%}
```

## 4. Date formatting

```liquid
{{ 'now' | date: '%B %-d, %Y' }}                         → September 28, 2026
{{ 'now' | time_zone: ${time_zone} | date: '%-I:%M %p' }} → local time of day
```

Adding days: convert to epoch seconds, add, format.

```liquid
{%- assign expires_ts = 'now' | date: '%s' | plus: 604800 -%}   {%- comment -%} +7 days {%- endcomment -%}
{%- assign tz = ${time_zone} | default: 'America/Chicago' -%}
{%- assign expires_on = expires_ts | time_zone: tz | date: '%A, %B %-d' -%}
Your code expires {{expires_on}}.
```

Days since a stored time:

```liquid
{%- assign last_ts = custom_attribute.${last_order_at} | date: '%s' | plus: 0 -%}
{%- assign now_ts = 'now' | date: '%s' | plus: 0 -%}
{%- assign days_since = now_ts | minus: last_ts | divided_by: 86400 -%}
```

Guard a blank time before doing math on it (a blank date can evaluate as the epoch or as now, depending on the filter chain):

```liquid
{%- assign last_raw = custom_attribute.${last_order_at} -%}
{%- if last_raw == blank -%}{% abort_message('last_order_at missing') %}{%- endif -%}
```

Preview with test users in at least two time zones before launch.

## 5. Conditional content

```liquid
{%- assign source = canvas_entry_properties.${signup_source} | default: 'general' | downcase -%}
{%- case source -%}
  {%- when 'checkout' -%}Thanks for signing up at checkout.
  {%- when 'popup' or 'footer' -%}Thanks for signing up.
  {%- else -%}Thanks for joining Northwind.
{%- endcase -%}
```

Liquid has no parentheses in conditions and evaluates `and` / `or` right to left. `a or b and c` means `a or (b and c)`. When mixing them, nest `if` blocks instead:

```liquid
{%- if has_code -%}
  {%- if is_eligible or is_vip -%} ... {%- endif -%}
{%- endif -%}
```

Numbers from properties may arrive as strings. Coerce before comparing: `{%- assign total = canvas_entry_properties.${cart_total} | default: 0 | plus: 0 -%}`.

## 6. Whitespace control

`{%-` and `-%}` strip the whitespace (including newlines) on that side of a tag. Use them on every `assign`, `if`, `for`, and `comment` in HTML email so logic does not leave blank lines, stray spaces inside links, or broken table rows. Use them always in SMS and push, where every character counts against the limit.

```liquid
<ul>
{%- for line in cart_items limit: 5 %}
  <li>{{line.name | strip | escape}}</li>
{%- endfor %}
</ul>
```

Trim the tag side you do not want; keep a plain `%}` where you want the output's own newline preserved.

## 7. Promotion codes

Pull a code only on the branch that will use it: codes are consumed when the message sends, and a `promotion` tag on a branch that is not taken does not run.

```liquid
{%- if eligible -%}
  {%- capture code -%}{% promotion('northwind_cart10') %}{%- endcapture -%}
  {%- assign code = code | strip -%}
  {%- if code == blank -%}
    {% abort_message('promotion list northwind_cart10 empty') %}
  {%- endif -%}
  Use {{code}} at checkout.
{%- endif -%}
```

When two messages must show the same code, pull it once in a User Update step and store it on the profile (`UPD_WELCOME_S4A` in program 01), then read the attribute in both messages and abort if it is blank.

## 8. Currency formatting

Prefer a preformatted string from your catalog or API (`price_display`: `"$49.90"`). When you must format a number:

```liquid
{%- assign cents = canvas_entry_properties.${cart_total} | default: 0 | times: 100 | round -%}
{%- assign dollars = cents | divided_by: 100 -%}
{%- assign rem = cents | modulo: 100 -%}
{%- if rem < 10 -%}{%- assign rem = rem | prepend: '0' -%}{%- endif -%}
${{dollars}}.{{rem}}
```

`round: 2` alone renders `49.9`, not `49.90`.

## 9. Links

URL-encode every dynamic value in a URL, and check whether the base URL already has a query string:

```liquid
{%- if product_url contains '?' -%}{%- assign sep = '&amp;' -%}{%- else -%}{%- assign sep = '?' -%}{%- endif -%}
<a href="{{product_url}}{{sep}}utm_source=braze&amp;utm_medium=email&amp;utm_campaign=EM_BROWSE_S1">View</a>
<a href="https://www.example.com/cart?cid={{cart_id | url_encode}}">Cart</a>
```

Escape user-generated or catalog text rendered into HTML: `{{review_text | escape}}`.

## 10. Global footer

Every promotional email renders `CB_GLOBAL_FOOTER`, which carries the unsubscribe link, preference center link, and postal address:

```liquid
<p>Northwind Supply Co. &middot; 100 Example Street &middot; Springfield, ST 00000</p>
<p><a href="{{preference_center.${northwind_preferences} | default: 'https://www.example.com/preferences'}}">Email preferences</a> &middot; <a href="{{${set_user_to_unsubscribed_url}}}">Unsubscribe</a></p>
```

Transactional emails use `CB_TRX_FOOTER` (no marketing unsubscribe, a link to account settings instead). Confirmation emails in program 02 use `CB_GLOBAL_FOOTER_MINIMAL`.

## What not to do

- **No nested Connected Content.** Do not put a `connected_content` call inside a content block that is rendered inside a loop, and do not build one call's URL from another call's output inside a loop. Each nesting multiplies calls per recipient and hides failures. Make one call that returns everything, then loop over the result.
- **No Connected Content per item in a loop.** Ten cart lines should be one cart call, not ten product calls. Use a catalog lookup or a batch endpoint.
- **Always `url_encode` IDs and dynamic values in URLs.** An ID with a space, `&`, `#`, or `+` silently breaks the link or points it at the wrong record.
- **No `event_properties` in scheduled sends.** Scheduled campaigns and canvases have no triggering event; the tag renders blank. Use profile attributes.
- **No `event_properties` in later canvas steps.** In a canvas, use `canvas_entry_properties` for the entry event. `event_properties` works only right after the Action Path that captured the event.
- **No filters on blank values without a default.** `{{ blank_value | truncate: 20 }}` renders nothing; `{{ blank_value | plus: 1 }}` renders 1. Default first, then transform.
- **No `default` on booleans you care about.** `default` treats `false` as empty, so `{{ false | default: true }}` renders `true`. Compare booleans explicitly: `{% if flag == true %}`.
- **No API keys in Liquid.** Use stored Connected Content credentials.
- **No stale prices.** Event properties capture the price at event time. Render prices from the catalog or a live API at send time, or leave them out.
- **No silent failure.** If a block cannot render its core content, abort with a message that names the step and the reason, so the abort shows up clearly in the message activity log.
- **No `capitalize` on names.** It can lowercase everything after the first letter.
