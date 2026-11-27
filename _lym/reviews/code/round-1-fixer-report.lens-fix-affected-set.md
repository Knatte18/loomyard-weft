```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking whether the round-1 fixes cover every member of each set their findings rest on.

**Findings left unchecked** (no search rerun): F2, F5, F6, F7. Each rests on a single site, and my context shows no further member of any of them. F1, F3 and F4 were checked.

**Findings**

1. **LOW.** Changed text: the `# Guard step` section of `internal/gatecli/doc.go`, "It covers … outside the scope, so a narrower gate keeps full guard coverage. Under `--card` it also covers a scope package's guard test whose tier no listed tier is or covers."
   - The computed and `--base` scopes still drop guards the way F1 describes. `targetsFor` (`internal/impactset/targets.go`) runs `if len(tests) == 0 || set[graph.byDir[dir].ImportPath] { continue }`, whatever the guard's tier.
   - So a tier0 diff-scope call over an impacted `cmd/lym` runs its tier1 guards neither in the listed run nor in the guard step.
   - The new `--card`-only sentence reads as though the other narrowed scopes have no such gap. That contradicts `pattern/PATTERN-test-speed.md:120`, "runs every marked test by name under its file's tier, whatever the impacted set".
   - The behaviour predates this change and the fix did not make it worse. The fixer-report should name it, or the doc should state it as a bound.

2. **LOW.** Unchanged text the fix reaches: the change description, `_lym/landing/summary.md`. F1's `fix` key asked for its guard-step sentence to be updated, and it was not.
   - "The vet step, the watched-test skip and the guard step all work from the widened set." omits that a scope package's unlisted-tier guards stay in the guard step.
   - "the gate logs the cause and the scope holds the cards' packages alone" omits the new stderr line (F2).
   - Under `## Tests`, "`TestDependentPackages` and `TestCardDependents` … cover an importer and a tier-tagged test importer being added" is stale. F6 removed `TestCardDependents`' success row, and the test is now `TestCardDependents_UnlistableModuleKeepsTheCardsPackagesAndSaysSo`.
   - The file is under `_lym`. If the fixer treats it as outside its write surface, F1's disposition should be `fixed-differently` or name this part as left undone, with that reason.

**Sets with no gap found**
- **F3:** every guard-step sentence of the implementer stencil (the sanctioned-deviation list and step 3) now covers a guard failing in a listed tier's run. The plan spec's verify model and `docs/overview.md` state no rule that contradicts it.
- **F4:** the search for "card package" wording found no remaining site that means the narrow set where the widened one runs.

**Tool calls**
1. Grep for "card package" wording across code, docs and stencils (F4's search). My context held only the `doc.go` and `scope.go` sites.
2. Grep for guard-step wording, plus guard handling in `internal/impactset/targets.go` (F1's search). `targets.go` was not in my context.
3. Read spans of `docs/overview.md`, `contracts/specs/loom-plan-spec.md` and `targetsFor`. None of them had been shown.
