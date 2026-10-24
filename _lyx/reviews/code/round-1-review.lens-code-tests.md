```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it. The amendment's only new change is `docs/overview.md`, and the parent changed no `internal/gatecli` test file (its diff stat there is empty).

**Per behaviour: pinning test**

- **Vet runs are planned after the listed tiers and before the guard group, one per tag tier.**
  `TestPlanTierRuns`, row "a --card scope vets each tag tier no listed tier runs…" (`tiers_test.go`).
  The integration row "--card vets a card's packages per module under the tag of a tier that adds a file…" (`scopes_integration_test.go`) pins the real command line and the per-module directories.
- **No vet run for a tier a listed tier covers or runs.**
  `TestPlanTierRuns`, row "a --card scope vets no tier a listed tier covers".
- **No vet run under the `none` scoper.**
  `TestPlanTierRuns`, row "a --card scope under the none scoper plans no vet run".
- **No vet run for other scope forms.**
  No dedicated row. The computed, `--base`, `--all` and targets rows keep their expected run lists, and `tagTiers` is set only on the card branch, so an extra vet run there would break them.
- **`tagTiers` and `tagAddsFile` add a tier only when its tag adds a file to a card package.**
  Covered through `computeScope` by the integration rows: tier1 is vetted when a tier1 file exists. A tier whose tag adds no file is pinned only indirectly, by the vet args list in the first integration row naming tier1 alone.
- **A vet failure exits non-zero, the summary line has scope form `vet`, and the closing line names `vet tier1`.**
  Integration row "--card fails a card whose tier1 file uses a helper only a tier0 file defines…". It also checks the run record's tier list and that the guard step still runs after the failed vet.
- **The vet run takes part in the green record and carries.**
  Integration row "--card carries a vet run that passed on the same clean tree".
- **`runLabel` and `tierNumbers` handle the vet step.**
  `runLabel`'s `vet tier1` is pinned by the failure row. `tierNumbers` excluding vet runs is pinned indirectly: the carry row's tier-list check on the vet run record expects `[0]`.
- **`realGo` doc comment change (`test_integration_test.go`).**
  A comment only; no behaviour.

**Test economy**
No findings. I found no duplicate test, no per-helper test (`tagTiers` and `tagAddsFile` have no direct tests), and no behaviour left uncovered.

Tool calls: 1. `git diff` of the three gatecli test files, plus a stat of the parent's test changes, because the manifest held no test diff.
