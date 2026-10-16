# Card 1 — production-fans

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent`

**Intent:**
Add the four production fans and their new lenses to the embedded `burler.yaml` template, so they reach every hub through `ResolveFan`'s per-name fallback without rewriting an operator's `burler.yaml` (D5, D6, D8; `PATTERN-config-layers`).
The dormant `standard`, `full`, `discussion` and `plan` fans and their lenses stay as they are.

The seven reviewer lenses, `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `affected-set`, `guard-reach` and `way-forward`, are each a procedure that starts its fork's final message with its own list, and keep the bench text, which ends in its own `Your list:` sentence.
Of the five fixer lenses, `fix-consistency`, `fix-claims` and `fix-way-forward` keep the twill-fixer bench's lens text as it is, with no list sentence, since the fixer's fork boilerplate returns findings only;
`fix-affected-set` ends by listing each finding it left unchecked, as D8 requires;
and `fix-proposal` is a procedure with no list sentence.
No new lens carries the emphasis line, since per D4 a fork reports only what its lens's procedure produces:

- `interactions`: the bench's `i1-interactions` text (`docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/burler.yaml`).
- `existing-paths`: the bench's `i1-existing-paths` text, widened per D5: for each state, setting, file or invariant a decision introduces or changes, every existing path that writes, resets, reapplies or reads it, and for a reader whether it still reads the state right under the new behaviour; a path that undoes or misreads a decision is a design finding about that decision, never a missing call-site list.
- `bound-breaker`: the bench's `i1-bound-breaker` text.
- `walkthrough`: per D5, for each decision one concrete case executed step by step with the data each step is handed by the step before, the end state checked against the decision's own stated purpose; its list names each decision, the case used, and purpose met or not.
- `affected-set`, `guard-reach`, `way-forward`: the three Plan-Review lenses of D6, each with the list D6 names.
- `fix-consistency`, `fix-claims`, `fix-way-forward`: the twill-fixer bench's `consistency`, `claims` and `way-forward` lens texts (`docs/research/burler-twill-fixer-bench-2026-10-16/setup/selfcheck-block.md`), scoped to the change set.
- `fix-proposal`: per D8, for each finding fixed as proposed, the concrete case where the proposed change is wrong or what it breaks elsewhere in the artifact.
- `fix-affected-set`: per D8, one search per set-premised finding to check the fix covers every member; when those findings exceed the fork's call cap it reruns the most severe first and lists each finding it left unchecked.

New fans, in this lens order:

- `review-discussion`: `interactions`, `existing-paths`, `bound-breaker`, `walkthrough` (no `generic`).
- `review-plan`: `affected-set`, `guard-reach`, `way-forward`.
- `fix-discussion`: `fix-consistency`, `fix-claims`, `fix-proposal`, `fix-way-forward`.
- `fix-plan`: `fix-consistency`, `fix-claims`, `fix-proposal`, `fix-affected-set`.

`TestConfigTemplate_DecodesAndIsSelfConsistent` raises its `wantLenses` and `wantFans` pins by the lenses and fans this card adds, adds the four new fans to its per-fan entry-count map and to its `wantOrders` map with the lens order listed above, which resolves each against a zero `Config` through `ResolveFan` and so fails on a missing fan, a misordered fan or an undefined lens, and keeps its defined-lens cross-reference check.
Its emphasis-suffix assertion skips the lenses the four new fans name, per the bench-text choice above, while its `ignore ` check still covers every lens.
The header comment of `internal/burlerengine/template.yaml`, from "Lens prose is always phrased as EMPHASIS" onward, is edited to say the dormant lenses are emphasis prose and the procedure lenses of the four production fans end in their own list instead, while the no-"ignore" rule still holds for every lens.

**ImpactSummary:** template data, its header comment and the template self-consistency test's pins in `internal/burlerengine`; no fan any config names today changes, so no caller is affected.
