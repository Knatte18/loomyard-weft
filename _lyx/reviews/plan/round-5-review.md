---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/15-configsync-layer-reconcile.md:89
    summary: Card 15 changes what a clone's prime carries but leaves three integration-tagged clone tests that read per-worktree files from a clone unlisted or with no stated expectation, so the card fails its own verify.
    fix: "In card 15, add `internal/fabriccli_test#TestRunCLI_CloneEndToEnd` and `internal/fabriccli_test#TestRunCLI_AnchoredHubScenario` under **Edit:** (`TestRunCLI_CleanHubScenario` is already there), and extend the paragraph at line 89 with each new expectation: `TestRunCLI_CloneEndToEnd` stops asserting that `configengine.ConfigFile(weftBase, \"loom\")` exists and asserts that the board carries loom's repo default instead, keeping its check that no per-worktree `board.yaml` sits under the weft base; `TestRunCLI_AnchoredHubScenario`'s `CloneConfigCommit_AnchorScoped` step keeps its clean-prime check and asserts `git ls-files` on the board lists every per-worktree module's repo default, not `backend/_lyx/config/<m>.yaml` in the prime; `TestRunCLI_CleanHubScenario`'s `CloneConfigCommit_PairInheritsConfigs` step asserts a pair forked from the clone resolves loom from the board's repo default (for example, `loomengine.LoadConfig` over `fabricengine.ConfigDirs` of the pair loads with no worktree file present) instead of asserting an anchored `loom.yaml` on the pair's weft sibling. Card 16 then adds to the same `PairInheritsConfigs` step that the pair inherits the sparse marker. Make the sweep method explicit too: the paragraph at line 89 names the search it ran (every test under any build tag that reads `configengine.ConfigFile` or `ConfigFileRel` of a per-worktree module from a `hubforge` hub or a `CloneAndWire` result without seeding it first), so a later round can rerun it."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/15-configsync-layer-reconcile.md:82
    summary: Card 15's reseed case in TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree, run under card 16's newStaleHub, makes the prime and pair file of the reseeded module strip to empty and be deleted; neither card names that module or that outcome, and if it is batcher, retiredKeyPresent fails the test.
    fix: "In card 15's line 82, name the module whose repo default the case deletes, and make it any per-worktree module other than `batcher` (for example `loom`). In card 16's paragraph at line 69, add the case's post-migration expectation: the deleted default is reseeded from the prime's file, which card 16's `newStaleHub` made full, so that module's prime and pair files equal the reseeded default and the strip deletes them; the case asserts both files are absent after the walk, and the deletion is committed in each worktree's one reconcile commit. `retiredKeyPresent` keeps reading `batcher.yaml`, which the case leaves with its default intact."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/16-sparse-migration.md:12
    summary: The hubreconcile test edits are not named as member glyphs, card 15 lists only the test file path while its Intent changes a named test, and card 16 changes the helper runStaleBuildReconcile without listing it.
    fix: "In card 15's **Edit:**, replace `internal/hubreconcile/ensure_integration_test.go` with `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` and `internal/hubreconcile_test#runStaleBuildReconcile`, the members its line 82 changes. In card 16's **Edit:**, add `internal/hubreconcile_test#runStaleBuildReconcile` beside `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, since its line 69 has that helper remove the stamp and assert the second walk."
---

### [BLOCKING] Clone tests that read the prime's or a pair's per-worktree files are not carried

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md:89`

**Issue:**
Card 15 makes `ReconcileAll` in a hub add no key and leave an absent worktree file absent, and its line 71 says a clone's prime carries only the per-worktree files the records repository already held.
The hubforge weft bare is empty (`internal/hubforge/hub.go`, `buildBareTemplate`'s doc), and so is the `TestRunCLI_CloneEndToEnd` fixture (its comment at `internal/fabriccli/cli_test.go:136-139`).
After card 15, then, a clone's prime and every pair forked from it hold no per-worktree config file.
Three existing tests, all under `//go:build integration` and so run by the plan's `go test -tags integration ./...`, still expect those files, and card 15 does not carry any of them:

- `internal/fabriccli/cli_test.go:177-180` (`TestRunCLI_CloneEndToEnd`) fails with `per-worktree loom config missing at …` when `configengine.ConfigFile(weftBase, "loom")` is absent.
  No card lists it.
- `internal/fabriccli/cleanhub_integration_test.go:522-534` (`TestRunCLI_AnchoredHubScenario`, step `CloneConfigCommit_AnchorScoped`) asserts that `git ls-files` in the weft prime holds `backend/<ConfigFileRel(m)>` for every per-worktree module.
  No card lists it.
- `internal/fabriccli/cleanhub_integration_test.go:360-368` (`TestRunCLI_CleanHubScenario`, step `CloneConfigCommit_PairInheritsConfigs`) asserts that a pair forked from the clone has `loom.yaml` on its anchored weft sibling.
  Cards 15 and 16 list the function, but card 15's lines 83-87 restate the new expectation of three other steps only, and card 16's line 56 adds only marker assertions, so this step's change is left unstated.

The other tests I checked read a per-worktree file only after seeding or committing it first, or tolerate its absence:
`hubforge` `TestNewHub` and `orchcli` `TestOrchIntegration_Refusals` (both carried by card 15),
`battencli` `TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating`,
`preflightshed`'s and `loomcli`'s retired-batcher tests,
`landingshed`'s `readParentLoomConfig`,
`webstercli`'s `makeConfigUnreadable`, whose callers seed `batcher` first,
and `configcli` `TestConfigOverRealHub`.
The gap is one enumeration method rather than three separate misses, so it is filed once.

### [MEDIUM] The reseed case and the migration fixture interact silently

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md:82`

**Issue:**
Card 15 adds a case to `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`: a stamp naming the running build, beside one deleted per-worktree default, and `Ensure` reseeds it.
By card 15's line 57, an absent default is seeded from the prime's file when that file exists and parses.
Card 16's `newStaleHub` (line 61) gives the prime and the pair the same full file per module: the template plus one key that differs from the repo default.
In the reseed case, the reseeded default is therefore the prime's own file, reconciled.
Card 16's strip then finds every key of that module's prime and pair files equal to the layer below, empties them and deletes them.
If the deleted default is `batcher`'s, the prime's `batcher.yaml`, which holds the template, the retired key in `orientation`'s place and the differing key, ends up with every key equal to the reseeded default once `orientation` is re-added and the retired key pruned.
It is deleted, and `retiredKeyPresent` (`internal/hubreconcile/ensure_integration_test.go:89-97`) fails with `read batcher.yaml`.
That contradicts card 16's line 64 claim that `batcher.yaml` survives the strip non-empty.
Neither card names the deleted module or states this case's expectation after migration.

### [MEDIUM] hubreconcile test edits are not named as member glyphs

**Location:** `_lyx/plan/16-sparse-migration.md:12`

**Issue:**
Card 15's **Edit:** lists `internal/hubreconcile/ensure_integration_test.go` as a bare file path, while its line 82 changes the named test `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`.
Card 16's line 69 changes the helper `runStaleBuildReconcile`, which removes `geom.StampPath()` before the second `Ensure` and asserts no new commit and byte-identical files, but its **Edit:** lists only the test function, not the helper.
The rubric's member-glyph rule asks for the changed members to be named.

## Focus settled

- Card 15, tests reading per-worktree files from a hubforge hub or a clone: not settled in the plan's favour; see F1.
- Card 16, `newStaleHub`: each `Ensure` test built on it starts from an unmarked prime and pair with a full per-worktree file set and the retired key present.
  The file set is written first and `seedRetiredKey` runs after it, adding the differing key, so `batcher.yaml` keeps one non-default key through the strip and `retiredKeyPresent` reads a non-empty file.
  That holds for every test built on `newStaleHub`, including `TestEnsure_PairCallReconcilesDespiteAFreshStamp`, `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` after its repair, and `TestEnsure_ConcurrentCallsLandOneCommitPerBranch`.
  The one exception is card 15's reseed case (F2).
- Card 16, second walks: `runStaleBuildReconcile` removes the stamp, so its second `Ensure` walks, and the marked worktrees make its no-commit and byte-identical assertions meaningful.
  In `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`, the first `EnsureWithHooks` skips the pair and writes no stamp, and the clear func removes the merge record.
  The plain `Ensure` that follows therefore walks the still-unmarked pair, so its strip-and-mark assertion tests something.
  Not a defect.
