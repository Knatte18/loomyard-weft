# Card 5 — stencils-and-spec

**Prosa:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/specs/loom-plan-spec.md`

**Edit:**
- `contracts/stencils#wordingClaims`

**Intent:**
Tell the agents about the verbs and the reachability rule (D4, D5), one sentence per line.

- `loom-template-plan.md`'s "On-disk layout" section, `loom-template-rework.md` beside its card-numbering bullets, and `loom-rubric-plan-review.md`'s `## Write surface` section each gain one sentence: a card is inserted or removed only with `lyx loom insert-card` or `lyx loom remove-card`, which renumber the later cards and whose `--help` says what they do, never by renaming, rewriting under a new name or deleting a card file by hand.
  The sentences restate neither the bounds nor the envelope (`PATTERN-producer-pointer-rule`).
- `loom-template-plan.md` gains, after its attack-surface paragraph, the writer rule: a card that prescribes a refusal or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes, and the test that reaches it; a propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt; a branch that upstream validation already rules out is not prescribed, but dropped or, when it guards an invariant, written as a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no per-row test; and a test that calls only the refusal's error constructor does not reach it.
- `loom-rubric-plan-review.md`'s `## Also flag` gains the matching bullet, "Reachability of a prescribed refusal", before the writer/reviewer symmetry note: a card prescribing a refusal or guard branch with no named reaching input or test is a finding, and so is one whose named input an upstream check rejects, the finding naming that check; the same raw-I/O and wiring-guard exemptions; a constructor-only test does not reach a refusal.
- `contracts/specs/loom-plan-spec.md`'s "Numbering and commit subject" section gains that numbering changes only through `lyx loom insert-card` and `lyx loom remove-card`, which move every later card's file name prefix, heading number, explicit `Commit:` prefix and Card Index number, and report prose mentions of a moved number without rewriting them.

`wordingClaims` in `contracts/stencils/claims_test.go` pins each new sentence with a short distinctive phrase: `lyx loom insert-card` in the plan template, the rework template and the rubric's `## Write surface` section (scoped with `inSection`), and in the plan template and the rubric `names in its` or the rubric's bullet title, `upstream check` and `error constructor`, so a reworded stencil fails `TestStencilClaims`.

**ImpactSummary:** prose only in three stencils and one spec, plus claim rows in the existing stencil claims table; no Go behavior changes.
