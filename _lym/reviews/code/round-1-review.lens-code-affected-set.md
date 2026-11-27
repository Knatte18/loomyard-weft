```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: what the diff's changes force outside the lines it touches.

**Per change, the set found, covered or not**

1. **New check id `create-caller-later`, wired into `validate` (`internal/planparser/validate.go:94`).**
   - Readers are `planglyph.validateFormat`, `validate`, `dispatchFindings` (which drops findings on done cards) and `loomshed.ValidatePlan` / `checkPartPlan`. All of them consume findings generically, so they are covered.
   - No Go registry, severity map or check-id table needs an entry. `convertAll` converts every finding generically. Covered.
   - Spec entry in `contracts/specs/loom-plan-checks-spec.md:177`: covered.
   - Card-granularity rule in `contracts/specs/loom-plan-spec.md:363-364`: covered.
   - `contracts/specs/loom-plan-spec.md:44-45` is not covered. It still says only `uses-later-target` and `delete-before-reference` refuse a dependency on a later card, and that "every other dependency on a later card is the writer's to avoid and the plan reviewer's to flag". `create-caller-later` now refuses one too. That sentence is stale (finding 1).
   - Stencils `contracts/stencils/loom/loom-block-plan-cards.md:56-57` and `contracts/stencils/loom/loom-rubric-plan-review.md:58-59` state the same writing rule. A machine check behind it leaves them true, and the board entry asks for no stencil change. Covered, no finding.
   - Package doc `internal/planparser/doc.go`: it lists no per-check files, only "Validate (validate.go) implements the plan format's validation checks", and the new file matches how `redundancy.go` and `extends.go` sit outside that line. Covered.
   - `PATTERN-refusal-way-forward`: `contracts/specs/refusal-spec.md:38` says a validation verb's findings envelope is a verdict, not a refusal. So the finding needs no `way forward:` clause, and its neighbours in `validate.go` and `redundancy.go` carry none. Covered.
   - `PATTERN-code-conventions`: the check id is an inline string literal, as every neighbouring check's id is. The guard path is not hard-coded in code. Covered.
   - `PATTERN-test-only-code` background (`pattern/PATTERN-test-only-code.md`): it covers allowlist entries only, not plan cards, so nothing is forced. Covered.
2. **Fixture change in `internal/planglyph/glyphchain_integration_test.go:244-245` (func → type).**
   - The case name "create handle beside its package self glyph" and its `redundant` expectations still hold for a type handle, since `redundant-package-target` treats every handle kind alike. Covered.
   - It is not a loosened check: no assertion, tolerance or limit changed. Covered.
   - I found 24 other test files that build plans with `-&gt; \`func` handles. Whether any of them now trips the check is the verify gate's to report, so I raised nothing on them.
3. **Commit messages (`4aa85ab3a..HEAD`).**
   - `1d196138a` carries a `Root-Cause:` trailer that names the defect the diff removes: no plan check tied a created function to its first-caller card. Covered.
   - Its body's claim "runs in ValidateFormat and Validate under language go" matches `GoToolchain()` at `latercaller.go:33`. Covered.
   - No `Pattern-Departure:` or `Loosened-Check:` trailers are needed. Covered.
4. **Declared PATTERN changes:** the directive declares none, so there is nothing to land.

**Findings**

1. **LOW, consistency, `contracts/specs/loom-plan-spec.md:44`.** The spec's statement of which checks refuse a dependency on a later card still names only `uses-later-target` and `delete-before-reference`. `create-caller-later`, added in the same diff, also refuses a plan whose card depends on a later card, here for its first production caller. The sentence's "every other dependency … is the writer's to avoid and the plan reviewer's to flag" is now wrong for this case. Evidence: `contracts/specs/loom-plan-spec.md:41-45` and the new `contracts/specs/loom-plan-checks-spec.md:177-188`. Fix: add `create-caller-later` to that sentence.

Tool calls:
- Grepped `internal` and `cmd` for test files that declare `-&gt; \`func` handles: the context had no list of fixtures the new check could affect.
- Read the `uses-later-target` context in `contracts/specs/loom-plan-spec.md`, the middle of `internal/planparser/doc.go`, and `pattern/PATTERN-refusal-way-forward.md`: none of the three was in the context.
- Grepped `contracts/specs/refusal-spec.md` for its scope over findings, and counted `way forward:` in neighbouring planparser checks: needed to settle whether the way-forward rule binds a plan finding.
- Grepped `pattern/PATTERN-test-only-code.md` for plan or card rules: its background was not in the context.
