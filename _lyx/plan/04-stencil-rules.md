# Card 4 — stencil-rules

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Tell the plan writers and Plan-Review about the verbs and the reachability rule through their own stencils (D4, D5).
Each new sentence points at the verbs and their `--help`, restating neither bounds nor envelope (`PATTERN-producer-pointer-rule`); none uses the phrase "edit or write", which `TestStencils_NoEditRuleRestatement` reserves for the edit directive, and none cites `contracts/specs/refusal-spec.md` by path, which `TestStencils_NoBareCrossRepoCitations` refuses; they name `refusal-spec.md` bare.

- `loom-template-plan.md`'s "On-disk layout" section, after "cards run `1..M` with no gaps.", gains: "Insert or remove a card only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards and whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand."
- `loom-template-rework.md`'s Step 3 gains the same sentence as a bullet.
- `loom-rubric-plan-review.md`'s `## Write surface` section, after its follow-up-card sentence, gains: "A fix inserts or removes a card only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards and whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand."
  The rubric reaches the Plan-Burler fixer through its round prompt, so the burler stencils stay unchanged.
- `loom-template-plan.md`, after its attack-surface paragraph ("Plan-Review flags one left unbounded."), gains the writer rule:
  "A card that prescribes a refusal or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes, and the test that reaches it.
  A propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt.
  A branch that upstream validation already rules out is not prescribed: drop it, or, when it guards an invariant, write it as a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no per-row test.
  A test that calls only the refusal's error constructor does not reach it."
- `loom-rubric-plan-review.md`'s `## Also flag` list, after "The attack-surface question" bullet, gains the matching bullet **Reachability of a prescribed refusal.**:
  "A card that prescribes a refusal or guard branch with no named input that reaches it, or no named test that reaches it, is a finding.
  So is one whose named input an upstream check rejects; the finding names that upstream check.
  A propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt, and so is a fail-loud wiring guard in that file's sense, which takes no per-row test.
  A test that calls only the refusal's error constructor does not reach the refusal."

Pin the wording in `wordingClaims` (`contracts/stencils/claims_test.go`), whose rows for the three stencils already exist: add `must` claims for "`lyx loom insert-card`" and "`lyx loom remove-card`" in each of the three stencils, the latter scoped to the rubric's `## Write surface` section; for "names in its `**Intent:**` the input that reaches it" and "error constructor does not reach it" in `loom-template-plan.md`; and for "Reachability of a prescribed refusal" and "error constructor does not reach the refusal" in `loom-rubric-plan-review.md`'s `## Also flag` section, each with a `why` naming this task's decision.

**ImpactSummary:** Prose-only changes to three loom stencils plus claim rows in the existing wording table; no Go behavior changes, and the stencil scans' phrase and citation rules are respected by wording.
