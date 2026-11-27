```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix's sets (test-support packages, later-card dependency sites, `ValidateFormat` callers, "own func handle" statements, unpinned branches).

Findings left unchecked: none.

1. **LOW** — `_lym/landing/summary.md`, Merge danger "Blast radius: planparser, and every caller of its `ValidateFormat`: the plan gates (loomshed's `ValidatePlan` and the part check) and webster's dispatch validation".
   The text says "every caller" but its list leaves members out.
   The rework gate calls `index.ValidateRework` (`internal/loomshed/gates.go:448`), which runs `validateFormat`, and is neither `ValidatePlan` nor the part check.
   `internal/runplan/conflict.go:83` and `internal/webstercli/validate.go:124` call `ValidateDispatch`; "webster's dispatch validation" covers the second, and the first only loosely.
   `planglyph.Validate` also reaches the check through `planparser.Validate` (`internal/planglyph/planglyph.go:73`).
   Naming the group without "every ... :" followed by a partial list would fix it.

2. **LOW** — `_lym/landing/summary.md`, Design decisions: "Requiring a later card that names the function and can call it keeps such a card silent when its later users change only test code."
   The F5 change makes this false.
   A later card whose only change creates a new plain-head `func` now counts as able to call, even when that `func` is a second helper in a `_test.go` file (declared on the package directory, with no `testing.` in its head and no test-prefixed name).
   Such a card changes only test code, yet the helper-creating card is now refused where it was silent before.
   The same omission is in the new Limits bullet and in `contracts/specs/loom-plan-checks-spec.md`: "A later card that names it and changes production code raises the finding".
   A later card that only creates another helper raises it too.

3. **LOW** — `internal/planparser/latercaller_test.go`, F2/F3 coverage: two test-by-name branches still have no row of their own.
   These are `!hasTestPrefix(parsed.Name)` in `refCanHoldProductionCaller` and the `hasTestPrefix(identifier)` return in `isTestHandle`.
   The only test-member refs in the table (`internal/watch_test#TestLevel`, `internal/watch_test#helper`) sit in a `_test` unit, so the unit condition alone rejects them.
   A mutant that drops either name check survives.
   F3's issue asked for "a `_test` self glyph or test member as each condition alone".
   A row using something like `internal/dobby#TestTick` and `plan:internal/watch#TestHelper` as the later card's only refs would pin both.

The other sets are covered:
- `testSupportDirectories` matches all three entries of the guard's `testSupportPackages` (`cmd/lym/testonlycode_linux_test.go:35-39`). Nothing keeps the two lists in step.
- The later-card dependency sites naming these checks are `loom-plan-spec.md:44` and `:364`, and both agree.
- The "own `func` handle" statements in the checks spec, the summary and the doc comments were all updated.

Tool calls (4):
1. grep for `uses-later-target` and `create-caller-later` across the repo, to find F4's sibling sites; my context held only some spec spans.
2. grep for callers of the validation entry points, for F6's set; my context held only a partial caller grep.
3. grep for the test-support package lists, for F1's set; my context held the guard's list but no other sites.
4. grep and read around `loom-plan-spec.md:364` and for "first production caller" in the specs and stencils; those lines were not in context.
