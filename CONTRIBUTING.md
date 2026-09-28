# Contributing

Thanks for improving the kit. Contributions that make a program more correct, more defensive, or easier to build are the most valuable. Contributions that add volume without adding judgment are the least.

## Adding a program

1. **Copy the template.** Create `programs/NN-<program-slug>/` using the next free number and a lowercase hyphenated slug. Copy [`programs/_TEMPLATE.md`](programs/_TEMPLATE.md) into it as `PROGRAM.md`.
2. **Fill every section.** Goal, Why it works, Entry, Audience, Message map, Guardrails, Measurement. If a section genuinely does not apply, say why in one line. Do not delete it.
3. **Add the build recipe.** A complete program also has `BUILD.md` (numbered canvas construction steps plus program-specific QA additions) and `snippets.liquid` (one commented block per message step). A strategy-only contribution is welcome too: mark it 📝 Planned in the README table.
4. **Follow the taxonomy.** Every canvas, message, segment, content block, tag, and custom event name follows [`library/naming-taxonomy.md`](library/naming-taxonomy.md). Reuse existing segments from [`library/segment-recipes.md`](library/segment-recipes.md) before inventing new ones; if you need a new one, add its definition there in the same pull request.
5. **Self-QA before you open the PR.** Walk [`library/qa-checklist.md`](library/qa-checklist.md) against your `BUILD.md` as if you were about to launch it. Run your Liquid past the checks in [`skills/braze-liquid-reviewer/SKILL.md`](skills/braze-liquid-reviewer/SKILL.md), by hand or with the skill. Note in the PR description anything you could not verify.
6. **Update the README table** with the program row and status.

## Ground rules

- **Fictional data only.** Use Northwind Supply Co., `example.com` domains, and invented product names. No real customer data, segment IDs, workspace IDs, API keys, app group identifiers, promotion code lists, or anything from an employer or client. If you are unsure whether something is identifying, leave it out.
- **Generic tactics, not proprietary playbooks.** Share patterns that any competent lifecycle team could arrive at. Do not contribute a specific company's internal strategy, results, or creative.
- **One program per folder.** A folder holds one canvas (or one tightly coupled canvas pair, like double opt-in). Shared logic goes in `library/`, not copied between programs.
- **Defensive Liquid.** Every output tag has a fallback. Every Connected Content call has a status check and a static fallback. Every loop in email strips whitespace. Any message that cannot render meaningfully without its data aborts rather than sending something broken.
- **Every program has a control group and guardrails.** 10% holdout by default, unsubscribe under 0.3% per send, complaints under 0.1% per send. Transactional messages are the only exception and must say so.
- **Incentives are fenced.** Any discount step has a margin fence and excludes `discount-abusers`. No escalating discount ladders.
- **No unsupported claims.** Targets are labeled as starting hypotheses. Do not cite results without a public source.
- **Skills reference the library.** A skill links to the checklist or taxonomy it applies. It does not paste a copy that will drift.
- **Plain writing.** Short sentences, active voice, no hype.

## Pull request checklist

- [ ] Folder and file names follow the structure in the README
- [ ] Every `PROGRAM.md` section is filled
- [ ] All names follow the taxonomy
- [ ] Liquid has fallbacks and failure handling throughout
- [ ] Control group and guardrail metrics documented
- [ ] No real brands, IDs, keys, or personal data (search your diff before you push)
- [ ] README program table updated
