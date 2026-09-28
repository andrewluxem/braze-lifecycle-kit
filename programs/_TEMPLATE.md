# NN · Program Name

> **Status:** 📝 Planned | ✅ Complete
> **Canvas:** `CANVAS | <Program> | v1`
> **Owner:** <role, e.g. Lifecycle Manager>
> **Related programs:** <numbers and names of programs this one hands off to, competes with, or must outrank>

<!--
How to use this template
- Copy to programs/NN-<slug>/PROGRAM.md. Fill every section; if one does not apply, say why in one line.
- Names follow library/naming-taxonomy.md. Segments come from library/segment-recipes.md.
- Keep it fictional: Northwind Supply Co., example.com.
- A complete program also ships BUILD.md and snippets.liquid. Delete this comment when done.
-->

## Goal

<One sentence: the behavior change you want, for whom, and in what window. Example: "Convert new subscribers into first-time buyers within 30 days of signup.">

## Why it works

<The behavioral logic in two to four short paragraphs. What state is the customer in at entry? What question or friction is stopping the next step? Why does this timing and sequence answer it? Name the mechanism (intent decay, loss aversion, social proof, habit formation, reciprocity) and say why it applies here, not just that it exists.>

## Entry

| Setting | Value |
|---|---|
| Entry type | <Action-based / Scheduled / API-triggered> |
| Trigger | <custom event, purchase, or schedule> |
| Trigger properties used | <`property_name` (type): what it drives> |
| Entry window | <start and end dates, or "always on"> |
| Re-eligibility | <never / N days / every schedule run> |

## Audience

**Target:** <who should enter, with segment names from the library>

**Exclusions:** <segments and filters that keep the wrong people out, with the reason for each>

**Exit criteria:** <events or filter changes that remove a user mid-canvas, and why>

## Message map

| Step | Timing | Channel | Name | Purpose | Key content |
|---|---|---|---|---|---|
| 1 | T+0 | Email | `EM_<PROGRAM>_S1` | <job of this message> | <what it says and shows> |
| 2 | T+Nd | Push | `PSH_<PROGRAM>_S2` | | |

**Per-step notes**

- **Splits:** <decision splits, audience paths, action paths and what each branch gets>
- **Quiet hours:** <which steps respect quiet hours and what happens when a send lands inside them>
- **Frequency caps:** <whether global caps apply, and any exception with its justification>
- **Incentives:** <depth, code type, expiry, margin fence, `discount-abusers` handling, or "none">

## Guardrails

- <Consent and subscription checks per channel>
- <Brand-safety rules: what the message must never say or imply>
- <Incentive fences, rate limits, kill switches>
- <Data dependencies and what happens when they fail>

## Measurement

| Item | Definition |
|---|---|
| Primary KPI | <metric, window, and starting target (a hypothesis to test, not a promise)> |
| Secondary metrics | <two to four supporting metrics> |
| Control design | 10% global holdout at canvas entry, <any variation> |
| Attribution window | <N days from entry, and why> |
| Conversion event | <the Braze conversion event and deadline> |
| Guardrail metrics | Unsubscribe <0.3% per send, complaint <0.1% per send, <program-specific guardrails> |
| Kill criteria | <what result, sustained for how long, pauses the canvas> |

See [`library/measurement-framework.md`](../library/measurement-framework.md) for holdout and reporting standards.
