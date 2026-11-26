```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced every change in the diff to the board entry's goal points and checked that each goal point has a change that builds it.

**Decisions.** The decision record holds no plan and no `Added after Discussion` entries, so the decisions are the goal points of the board entry:
- **G1 (#263):** the card gate also runs the packages whose tests depend on the changed packages, or on the changed strings.
- **G2 (#263):** plan review rejects a repository-wide intent whose edit list is a fixed set of packages with no step that proves the set complete.
- **G3 (#262):** either plan writing and plan review keep a new function and its first caller in one card, or the card gate tolerates a function whose caller a later card declares.

**Changes and what they trace to:**
- **C1 builds G1.** `impactset.DependentPackages` and `dependentsFor` (`internal/impactset/dependents.go`), plus the `impactset/doc.go` paragraph.
- **C2 builds G1.** `cardDependents` and the `--card` branch of `computeScope` (`internal/gatecli/scope.go:198`, `:282-293`), plus the `computeScope` doc line.
- **C3 is forced by C1 and C2.** The tests: `TestDependentsFor`, `TestDependentPackages`, `TestCardDependents` and the updated `TestGateTest_Scopes` rows (`scopes_integration_test.go:405,490,541,569,581`).
- **C4 is forced by C2.** Each of these edits rewrites a statement of the card scope that C2 made stale:
  - `internal/gatecli/doc.go:140-143`
  - the `--card` flag help (`cli.go:125`)
  - the `gate.yaml` template comment (`template.yaml:25`)
  - `docs/overview.md:960`
  - the plan spec's verify model (`loom-plan-spec.md:327,335`)
  - the implementer stencil (`webster-body-implementer.md:73`)
- **C5 builds G3, taking its first option.** The new-function rule in:
  - the plan-cards block (`loom-block-plan-cards.md:56-57`)
  - the plan spec's granularity section (`loom-plan-spec.md:362-363`)
  - the Plan-Review granularity bullet (`loom-rubric-plan-review.md:58-60`)
- **C6 builds G2.** The repository-wide-intent rule:
  - The Plan-Review bullet (`loom-rubric-plan-review.md:61-65`) is a finding rule under `## Also flag` (line 52), not under `## Do not flag` (line 36).
  - The writer-side copies in `loom-block-plan-cards.md:53-54` and `loom-plan-spec.md:365-366` are forced by C6. A review rule needs a matching writer rule, which the rubric's writer/reviewer symmetry note (line 132) requires.

**What builds each decision:**
- G1 is built by C1 and C2, with C3 and C4 forced by them. The goal point offers two routes joined by "or", and the diff takes the package-dependency route only. The change description states that choice under "Design decisions", so the diff still builds the goal point. Whether that route reaches the packages the entry reports is a correctness question for another lens and the handler, not a scope gap.
- G2 is built by C6.
- G3 is built by C5.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point has a change that builds it.

Tool calls:
1. `grep` of the section headings in `contracts/stencils/loom/loom-rubric-plan-review.md`. My context held only the diff hunk, whose header did not show whether the new bullets sit under `## Do not flag` or `## Also flag`.
