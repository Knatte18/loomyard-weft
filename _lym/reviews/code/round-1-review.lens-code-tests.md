```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke, and report any behaviour with no such row.

**Per behaviour, the test that pins it**

1. **`Config.Gate` in a hub with no tiers returns the key with no tiers and no refusal.** Pinned by `internal/gateslot/config_test.go:949`, the `TierList` row "a hub that declares no tier". It goes through `Gate`, and the same path serves `renderGateCall`, which is the shed-build render that failed in issue #133.
2. **`Config.Gate` in a hub with no tiers still refuses a missing key, because that check now runs first.** Pinned by `config_test.go:950`.
3. **A hub that declares tiers but no tier0 is still refused.** Pinned by `config_test.go:948`, the existing row "a hub with no tier0".
4. **An empty tier list is still refused.** Pinned by `config_test.go:946-947`, existing rows.
5. **In a hub with no tiers, `runTest --gate` exits 0, prints its line and runs no tier command.** Pinned by `internal/gatecli/test_integration_test.go:2202-2212`. It asserts exit 0, the exact line text and that the stand-in never ran.
6. **In a hub with no tiers, a call without `--gate` is still refused.** Pinned by `test_integration_test.go:2109-2114`, the existing row "a hub with no tier names lym config gate".
7. **`allScope`: a `--gate` key of scope `all` over a diff that holds only inert files gets the empty scope and names it.** Pinned by `internal/gatecli/scopes_integration_test.go:630-639` (`wantArgs` with no args, plus the two stdout strings).
8. **`allScope`: an explicit `--all` with no `--gate` stays whole over an inert-only change.** Pinned by `scopes_integration_test.go:907`, the existing "an --all run carries over a committed inert-only change…" test. If the `request.gate == ""` guard broke, that test's carried run and current-tree entry would change.
9. **`allScope`: a hub with no `inert_paths` stays whole.** Pinned by `test_integration_test.go:756-766`, a prime `--gate finalize` whose config has no `inert_paths` and which expects both tiers to run.
10. **`allScope`: a worktree with no parent branch, in a hub with `inert_paths`, stays whole.** No test.
11. **`allScope`: a merge base that cannot be read stays whole and prints its stderr line.** No test.
12. **The stencil wording in `darn-fragment-make-change-gated.md`.** Exempt as static prose.
13. **The `doc.go` changes in gatecli and gateslot.** Exempt as static prose.

**Findings**

- **LOW, design.** Location: `internal/gatecli/scope.go:337-343` (`allScope`). The diff adds two fall-back-to-whole branches, and no test row covers either:
  - a `--gate` key of scope `all` in a worktree with no parent branch, in a hub with `inert_paths`;
  - a merge base that cannot be read, which also prints a line on stderr.

  Evidence:
  - `test_integration_test.go:756` runs the prime `--gate finalize` with no `inert_paths`, so it returns at `scope.go:334` and never reaches `parentMergeBase`.
  - `scopes_integration_test.go:630` runs in `pair`, which has a parent branch.

  Fix: add a row in `scopes_integration_test.go` that runs `--gate finalize` from `prime` after `writeInert` with an inert-only edit, and asserts that the tier runs whole and the "scope is empty" line is absent.

Tool calls:
1. `config_test.go:925-975` and greps across the gatecli tests for scope-all gate calls and "declares no tier". My context had the diff hunk but not the `TierList` loop body, nor the existing tests that call scope-all gate keys.
2. `test_integration_test.go:2100-2122` and `750-770`, plus a grep for `inert_paths`. I needed to confirm that a call without `--gate` in a hub with no tiers is still refused, and what config the prime finalize test runs under.
