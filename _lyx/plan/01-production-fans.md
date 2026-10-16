# Card 1 — production-fans

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestResolveFan`

**Intent:**
Add the four production fans and their new lenses to the embedded `burler.yaml` template, so they reach every hub through `ResolveFan`'s per-name fallback without rewriting an operator's `burler.yaml` (D5, D6, D8; `PATTERN-config-layers`).
The dormant `standard`, `full`, `discussion` and `plan` fans and their lenses stay as they are.

New lenses, each a procedure that starts its fork's final message with its own list, written as emphasis, never exclusion, per the template's header:

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

`TestResolveFan` gains a row resolving each of the four names against a zero `Config` and asserting the lens names in exactly that order, which fails if a fan is missing, misordered or names an undefined lens.
`TestConfigTemplate_DecodesAndIsSelfConsistent` already checks every template fan names a defined lens and needs no change.

**ImpactSummary:** template data plus one test row in `internal/burlerengine`; no fan any config names today changes, so no caller is affected.
