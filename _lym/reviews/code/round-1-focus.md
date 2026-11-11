---
round: 1
focus:
  - "internal/gatecli/coveragescope.go: check the D3 union per importing package and tier (selected, unrecorded, out-of-process or unresolved, guard) against holdsTierTest host admission and the watch skip. Does an importer with an empty union and no miss run nothing, and does a changed or lym-binary package always run whole?"
  - "internal/gatecli/tiers.go and run.go: does the second selected invocation carry its anchored -run in the run's own args, not the call's, so the green entry never serves as a later base? Check the over-budget whole fallback, and how --stop-on-fail, the failed-test rerun and the verify budget act on both invocations."
  - "internal/gatecli/scope.go and coveragescope.go: does a coverage call with explicit targets (tree_scope, rerun, named targets) run un-narrowed? Does coverage under scoper none or dotnet equal diff, and can coverage ever add a package diff would not run?"
  - "internal/gatecli/coveragescope.go D8 and D5: are maps read only through testcover.ReadMap and never built, and is every read failure a miss that runs the package whole? Are file contents at a map's Commit read through fabricengine CodeFileAt and never a new git call?"
  - "internal/gatecli and internal/impactset: check the D7 label set and the per-package report against the criteria. Check impactset Targets keeping changed packages, importers and binary builders apart, with Packages unchanged for existing callers."
  - "internal/gateslot (config.go, tiers.go, template.yaml, doc.go): is coverage accepted by the strict loader and named in the scope error? Do darn-write, webster-fix and code-burler-fix default to coverage, pr-review stay diff, and does the template comment name coverage and its default keys?"
  - "Tests: for the new tests in coveragescope_test.go, tiers_test.go, targets_test.go and config_test.go, check each against the acceptance criteria for coverage gaps (guard, out-of-process, unrecorded, other-GOOS, over-budget, rerun, unreadable map). Apply test economy against the covering tests, since the redundancy report did not judge these."
  - "Scope and claims: compare the diff against the decision record. Flag changes to fabricengine/diff.go, loomcli/landingdeps_verify_test.go, testcover and pattern/PATTERN-test-speed.md that no decision licenses. Check the commit-message and doc.go claims against the code."
---

Seed focus for round 1, drawn from the decision record's D1, D3 and D6 to D8 and the acceptance criteria.
