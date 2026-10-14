# Card 5 — prescribed-refusal-reachability

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Make a prescribed refusal's reachability a plan-time judgment shared by the writer and the reviewer (decision record D5); no Go check is added.

`loom-template-plan.md` gains a paragraph directly after the attack-surface paragraph ("A card that introduces a new verb, flag, escape hatch or routing edge, ..." and "Plan-Review flags one left unbounded."), one sentence per line:

```markdown
A card that prescribes a refusal or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes, and the test that reaches it.
A propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt.
A branch that upstream validation already rules out is not prescribed: drop it, or, when it guards an invariant, write it as a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no per-row test.
A test that calls only the refusal's error constructor does not reach it.
```

`loom-rubric-plan-review.md`'s `## Also flag` gains a bullet after **The attack-surface question.**, with the same scope and exemption:

```markdown
- **Reachability of a prescribed refusal.**
  A card that prescribes a refusal or guard branch and names no input reaching it after every upstream check passes, or no test that reaches it, is a finding, and so is one whose named input an upstream check rejects; the finding names that upstream check.
  A test that calls only the refusal's error constructor does not reach it.
  A propagated raw I/O error that joins its section's grouped transient row in `refusal-spec.md` is exempt, and so is a fail-loud wiring guard in `refusal-spec.md`'s sense.
```

The writer rule and the rubric bullet are paired, so the rubric's writer/reviewer symmetry note holds.

`wordingClaims` gains, in the `loom-template-plan.md` row, `must` claims for "A card that prescribes a refusal or guard branch names in its `**Intent:**` the input that reaches it after every upstream check passes", "grouped transient row", "A branch that upstream validation already rules out is not prescribed", "fail-loud wiring guard" and "calls only the refusal's error constructor does not reach it", and in the `loom-rubric-plan-review.md` row, `section: "## Also flag"` claims for "Reachability of a prescribed refusal", "prescribes a refusal or guard branch", "the finding names that upstream check", "grouped transient row", "fail-loud wiring guard" and "calls only the refusal's error constructor does not reach it", so `TestStencilClaims` fails if either half loses its scope, either exemption or the constructor rule.

**ImpactSummary:** two stencils gain one paragraph or bullet each and the claims table eleven rows; no stencil marker, Go contract or other test changes.

**Commit:** 5: stencils: a prescribed refusal names its reaching input and test
