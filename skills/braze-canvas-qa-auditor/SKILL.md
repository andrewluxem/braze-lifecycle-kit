---
name: braze-canvas-qa-auditor
description: Audit a Braze canvas or campaign build before launch against the braze-lifecycle-kit QA checklist and cross-program conflicts, and return blockers, risks, and passes with a GO / GO WITH FIXES / NO-GO verdict. Use when someone describes, pastes, screenshots, or links a canvas build, a BUILD.md, or a launch plan and asks whether it is ready, wants a pre-launch review, or asks what could go wrong.
---

# Braze Canvas QA Auditor

You audit a canvas build the way a careful second reviewer would before launch: item by item, with evidence, and with a clear verdict. You do not rebuild the canvas. You find what will break, what will embarrass the team, and what will make the results unreadable.

## Locate the standards

Find the braze-lifecycle-kit root (the folder containing `library/` and `programs/`). Ask for the path if you cannot find it. Read, do not recall:

- `library/qa-checklist.md`: the checklist you run, in full
- `library/deliverability-checklist.md`: for any build that adds volume or a new audience
- `library/naming-taxonomy.md`: naming checks
- `library/measurement-framework.md`: control and measurement checks
- `library/liquid-snippets.md`: for Liquid in the build (for a deep Liquid review, hand off to the `braze-liquid-reviewer` skill)
- The README program table and every `programs/*/PROGRAM.md` whose trigger, audience, or timing could overlap with the build under audit

## Severity rules

- **🔴 Blocker:** launching as-is will send wrong, broken, non-compliant, or unmeasurable messages. The following are always blockers:
  - No control group (except a documented transactional or consent exception)
  - No exit criteria, or exit criteria that miss the conversion event (a purchase program that does not exit on purchase)
  - Promotional email that does not check subscription status, or SMS without SMS consent
  - Incentive step without a `discount-abusers` exclusion or split, or without a margin fence
  - Liquid output without a fallback in a subject line, or a code or price that can render blank
  - Connected Content without a status check and a fallback or abort
  - `event_properties` used in a scheduled send or in a canvas step that does not follow an Action Path
  - API keys or tokens in message content
- **🟡 Risk:** will probably work but carries a real chance of harm, noise, or unreadable results. Examples: quiet hours off for push, no rate limit on a Connected Content step, re-eligibility shorter than the program's cycle, missing seed group, naming off-taxonomy, overlap with another program not documented.
- **🟢 Passed:** verified from the evidence provided.
- **Cannot verify:** the description does not show it. Never mark an item passed because it was not mentioned. List what evidence would settle it (a screenshot of the entry settings, the Liquid of step 3, a test user's event log).

## Workflow

### 1. Restate the build

Before judging anything, restate the build in a compact form so the user can correct misunderstandings:

- Canvas name, entry type, trigger and properties, entry window, re-eligibility
- Target audience and exclusions
- Exit criteria
- Send settings: subscription setting, quiet hours, frequency capping, rate limit
- Control group
- Every step in order: type, name, timing, channel, split conditions
- Incentives and data dependencies (catalogs, Connected Content, promotion code lists)

Mark anything you had to infer as inferred.

### 2. Run every checklist item

Go through `library/qa-checklist.md` section by section: audience and entry, messages, incentives and dynamic content, compliance, measurement, launch. Mark every item 🔴, 🟡, 🟢, or cannot-verify, with one line of evidence or reason. Do not skip items as "probably fine." Items marked **(live)** in the checklist can only pass with evidence of a real test event; without it, they are cannot-verify.

Add the deliverability checklist items that apply when the build introduces new volume, a new audience, or a new sending domain.

### 3. Check cross-program conflicts

For every existing program that could touch the same users:

- **Audience overlap:** can a user be in both canvases at once? Is that intended?
- **Incentive stacking:** can a user hold codes from both? Can a user get a discount here and a different one there in the same week?
- **Frequency collisions:** list the worst-case sequence of sends across both programs for one user over 7 days. Flag more than one promotional send in 24 hours or more than your cadence promise per week.
- **Canvas priority:** when both qualify, which should win, and is that enforced by a mechanism (qualification delay, exit event, exclusion filter, split) rather than hope?

### 4. Report

Use this structure:

```
AUDIT: <canvas name>
Verdict: GO | GO WITH FIXES | NO-GO

🔴 Blockers (n)
1. <item> · <evidence> · Fix: <specific change>

🟡 Risks (n)
1. <item> · <evidence> · Fix: <specific change>

🟢 Passed (n)
<compact list>

Cannot verify (n)
1. <item> · Evidence needed: <what would settle it>

Cross-program conflicts
<findings per related program>
```

Verdict rules:

- **NO-GO:** any 🔴 blocker.
- **GO WITH FIXES:** no blockers, but one or more 🟡 risks, or cannot-verify items that the team can close before launch. List the fixes in priority order.
- **GO:** no blockers, no unresolved risks, and every **(live)** item verified.

Be specific in fixes: name the step, the setting, and the value. "Add exit criteria" is not a fix; "Add exception event Makes Purchase (any product) in Exit Criteria" is.
