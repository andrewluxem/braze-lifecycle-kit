# 12 · Re-permissioning

> **Status:** 📝 Planned (strategy brief; `BUILD.md` and `snippets.liquid` welcome as contributions)
> **Canvas:** `CANVAS | Repermission | v1`, plus a permanent preference-center link in `CB_GLOBAL_FOOTER`
> **Owner:** Lifecycle Manager
> **Related programs:** 02 Double Opt-In (sets the initial preferences), 08 Sunset Cleanup (the unengaged counterpart)

## Goal

Give engaged subscribers a regular, easy way to choose what they get and how often, and cut their unsubscribe rate by sending them only what they asked for.

## Why it works

Many unsubscribes are not "never again." They are "less of this" or "not that topic." When unsubscribe is the only control, people use it. A preference center with a small number of real choices (up to six topics and a frequency choice) gives them a middle option, and the choices become targeting data every other program can use.

Asking once at signup is not enough, because interests change. A quarterly check-in to engaged subscribers, framed as "make sure we're sending what you want," keeps the preferences fresh without pestering. A permanent footer link covers everyone in between.

## Entry

| Setting | Value |
|---|---|
| Entry type | Scheduled, quarterly (first Tuesday of January, April, July, October, 10:00 user local time) |
| Trigger | Membership in `email-engaged-90d` |
| Trigger properties used | None (scheduled). Profile attributes only. |
| Entry window | Always on |
| Re-eligibility | 80 days (once per quarter) |

## Audience

**Target:** `email-engaged-90d` subscribers who have not updated preferences in the last 90 days (`prefs_updated_at` older than 90 days or blank).

**Exclusions:** Subscribers who joined in the last 30 days (they just set preferences in program 01), `unengaged-180d` (program 08), `internal-test-users`.

**Exit criteria:** Submits the preference center (custom event `preferences_updated`).

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | Day 0 | Email | `EM_REPERM_S1` | Invite | "Pick what you want from us." The six topics as buttons, frequency choice, current selections shown, link to the full preference center. |
| 2 | Day 7, if opened and not submitted | In-App | `IAM_REPERM_S2` | Nudge app users | One-line prompt to the in-app preference screen. |

**Per-step notes:** No second email; this program exists to reduce email, not add to it. Quiet hours on. Frequency caps apply.

**Preference center design:** Up to 6 topics (Northwind: Gear guides, New arrivals, Sales, Camping, Climbing, Apparel), each a boolean custom attribute (`pref_topic_guides`, `pref_topic_new`, and so on); a frequency choice written to `pref_frequency` (`weekly`, `biweekly`, `monthly`); `prefs_updated_at` stamped on save. Every promotional campaign filters on the matching topic attribute and on `pref_frequency` through a frequency segment. A preference that no campaign reads is a broken promise; audit quarterly.

## Guardrails

- Unsubscribe from all stays one click away on the preference page and in every footer. The preference center is an alternative, never an obstacle.
- Topics must map to real content streams. Do not offer a topic you do not send.
- Preference writes go to custom attributes through the Braze preference center or your own page calling `/users/track`; never through a third-party form that drops the data.
- Default for a subscriber who never sets preferences: all topics on, `weekly`. Document it.

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | Unsubscribe-rate delta: 90-day unsubscribe rate of subscribers who completed the preference center vs matched subscribers in the 10% holdout |
| Secondary metrics | Completion rate, share choosing lower frequency, engagement rate per send for completers, share of unsubscribes that come through the preference page instead of the one-click link |
| Control design | 10% holdout does not receive the quarterly email (the footer link stays available to everyone) |
| Attribution window | 90 days |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send |
| Kill criteria | Completers unsubscribe at a higher rate than holdout for two quarters: the choices are not being honored downstream |
