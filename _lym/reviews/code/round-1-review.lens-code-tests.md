```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test that pins it.

**Per behaviour, the test that pins it:**
1. `burler-review-format.md:33-40`: the reviewer checks the facts a `fix` rests on (package clause and export status, the callee chain behind a claim that code only reads, whether named test infrastructure exists), uses the `decision` route when no fix holds, and marks an unchecked fact as unverified. No test pins this. The text is static prose an LLM reads, and it adds and changes no marker, so it is exempt.
2. `burler-step-fix.md:29-30`: the fixer reads a fact the `fix` names as unverified before applying the change. No test; static prose, exempt. The block adds no `{{.X}}` marker, so the fill's required-marker set is unchanged and no existing fill test is affected.
3. `loom-block-plan-cards.md:56-64`: the first-caller rule, the rule that a card's claims are read in the code, and the rule that cards do not contradict each other. No test; static prose, exempt. The block adds no marker, so its declared markers `specs_dir` and `gate_tiers` are unchanged.
4. `loom-rubric-plan-review.md:58-71`: three new "Also flag" bullets. No test; static prose, exempt. They add no marker beyond the ones `ReadRubric` fills.
5. `loom-plan-spec.md:366-368`: the first-caller exception added to card granularity. No test; static prose in a spec, exempt. No plan check enforces it, and the diff adds none.

**Findings:** none. Every behaviour the diff adds is stencil or spec prose, which the lens exempts. The decision record's added entry also rules out tests of stencil prose. The diff adds no test, so the test-economy rows have no subject.

**Tool calls:** none. The diff pieces and the full stencil files were already in context.
