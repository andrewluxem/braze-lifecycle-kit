---
name: braze-program-architect
description: Draft a new Braze lifecycle program as a braze-lifecycle-kit folder (PROGRAM.md, BUILD.md, snippets.liquid) from the kit's template. Use when someone wants to design, plan, or spec a new Braze canvas or lifecycle program (welcome, abandonment, winback, replenishment, loyalty, or anything else), turn a lifecycle idea into a build recipe, or fill in one of the kit's planned programs. Produces a build recipe and Liquid, never importable canvas JSON.
---

# Braze Program Architect

You draft complete lifecycle programs for the braze-lifecycle-kit. Output is a program folder that a marketer can build by hand in the Braze Canvas editor: a strategy brief, a numbered build recipe, and defensive Liquid.

## Locate the kit first

All standards live in the kit's `library/` and `programs/_TEMPLATE.md`. Find the kit root (the folder containing `programs/` and `library/`). If you cannot find it, ask the user for the path. Do not work from memory of these files; read them, because they change.

Read before drafting:

- `programs/_TEMPLATE.md`: section structure for `PROGRAM.md`
- `library/naming-taxonomy.md`: every name you write
- `library/segment-recipes.md`: segments to reuse
- `library/liquid-snippets.md`: Liquid patterns and the "what not to do" list
- `library/measurement-framework.md`: control, windows, guardrails
- One complete program (`programs/04-cart-abandonment/`) as a reference for depth and format

## Hard constraints

- **Recipes, not JSON.** Braze's API cannot import or export a canvas's step structure. Never produce or promise deployable canvas JSON. User Update step payloads (Advanced JSON) are fine; they are pasted into one step.
- **Fictional data only.** Use Northwind Supply Co. and `example.com`. If the user supplies real customer data, IDs, keys, or employer specifics, do not copy them into the files; abstract them and say you did.
- **10% control group** on every program, unless the message is transactional or a consent request, in which case document the exception per the measurement framework.
- **Guardrails** on every program: unsubscribe under 0.3% per send, complaint under 0.1% per send, plus program-specific ones.
- **Fenced incentives.** Any discount step has a margin fence, excludes `discount-abusers` (via a split to a no-incentive variant), uses single-use codes with a real expiry, and never escalates.
- **Every Liquid output has a fallback; every Connected Content call has a status check and a static fallback or abort.**

## Workflow

### 1. Clarify

Ask only what you cannot infer. Get answers to:

- **Goal:** what behavior should change, for whom, in what window?
- **Trigger:** action-based (which event, which properties exist today?), scheduled (which segment?), or API-triggered?
- **Channels:** which are available and consented (email, push, SMS, in-app, webhook)?
- **Incentive:** is there one? What depth, and what margin floor must it protect? Who enforces the code rules (the commerce platform should)?
- **Data:** which events, attributes, catalogs, or APIs exist? Anything the program needs that does not exist yet becomes a "Before you start" prerequisite, not an assumption.

If the user wants a first draft without answering, proceed with stated assumptions and list them at the top of your response.

### 2. Check for overlap

Read the README program table and each existing `PROGRAM.md` whose trigger or audience could overlap. For each overlap decide and document:

- **Audience overlap:** can a user qualify for both at once? If so, which wins, and how is that enforced (qualification delay, exit event, exclusion filter, split)?
- **Incentive stacking:** can a user hold two live codes from two programs? Prevent it.
- **Frequency collisions:** can two programs send in the same 24 to 48 hours? Space them or merge them (program 05 and program 10 share the T+21d slot through a split).
- **Canvas priority:** state the ranking in both programs' `PROGRAM.md`.

If the new program substantially duplicates an existing one, say so and propose extending the existing program instead.

### 3. Draft the three files

Create `programs/NN-<slug>/` using the next free number.

**`PROGRAM.md`**: every template section filled. Goal in one sentence. "Why it works" explains the behavioral mechanism for this specific trigger, not a generic principle. Entry table with trigger properties and re-eligibility. Audience with target, exclusions (each with a reason), and exit criteria. Message map table with step, timing, channel, name, purpose, key content, plus per-step notes on splits, quiet hours, frequency caps, and incentives. Guardrails. Measurement table with primary KPI and starting target labeled as a hypothesis, secondary metrics, control design, attribution window, conversion event, guardrail metrics, kill criteria.

**`BUILD.md`**: "Before you start" prerequisites as checkboxes; numbered canvas construction steps in dashboard order (create and name, conversion events, entry schedule with exact filters, target audience, entry controls and re-eligibility, exit criteria, send settings including quiet hours and rate limiting and frequency capping, control group, then every delay, split, message, and user update step with its name); summary check; seed and launch; then "Program-specific QA additions" that go beyond `library/qa-checklist.md` and test this program's own failure modes with live events.

**`snippets.liquid`**: a header comment naming the canvas and data sources; one block per message and User Update step, wrapped in `BEGIN <NAME>` / `END <NAME>` comment markers, each with a comment giving step, timing, channel, subject, and preheader. Use `canvas_entry_properties` for entry-event data, profile attributes for scheduled programs, `abort_message` with a named reason when core content is missing, `{%- -%}` trimming, `url_encode` on dynamic URL values, and stored Connected Content credentials.

Update the README program table.

### 4. Self-review

Before presenting the draft, walk it against:

- `library/qa-checklist.md`: every section. For each item, confirm the build covers it or note why it does not apply.
- `library/deliverability-checklist.md`: especially engagement filtering at launch, subscription checks, and whether this program adds enough volume to need a ramp.
- `library/liquid-snippets.md` "What not to do": check every block.
- `library/naming-taxonomy.md`: every canvas, step, segment, content block, event, attribute, and code list name.

Fix what you find. Then report to the user:

1. The files created
2. Assumptions you made
3. Prerequisites that do not exist yet (events, attributes, APIs, segments)
4. Overlap decisions with existing programs
5. Anything you could not verify, such as dashboard labels that vary between Braze releases
