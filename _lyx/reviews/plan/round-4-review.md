---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/15-configsync-layer-reconcile.md
    summary: Card 15 changes what a clone writes into the prime but plans no change to the existing tests that still expect the old clone output, so go test -tags integration fails after card 15
    fix: "In card 15, add `internal/hubforge#TestNewHub` and `internal/orchcli#TestOrchIntegration_Refusals` to **Edit:**. In its Intent, state the new expectations. TestNewHub's ConfigMaterializedWithoutSeeding step asserts the board's loom repo default matches the registered template, not the prime's loom.yaml. Its SeedConfigRedundantSeedDoesNotFatal step seeds loom once with SeedConfig before reading the file back. In TestRunCLI_CleanHubScenario (already a target), the CloneConfigCommit_WeftPrimeCleanAfterClone step asserts the prime is clean and that git ls-files on the board lists every per-worktree module's repo default. The CloneConfigCommit_OneCommitNotOnePerModule step asserts a fresh clone's prime carries no `fabric clone: record module configs` commit, and that a second ReconcileAll applies nothing. The `hub config reconcile` subtest of TestOrchIntegration_Refusals writes the unparseable pair loom.yaml without reading an original first, and removes that file afterwards instead of restoring the old bytes."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/16-sparse-migration.md
    summary: Card 16's newStaleHub writes a full per-worktree file set but leaves its order against the existing seedRetiredKey call open, and either order breaks the retired-key assertions
    fix: "In card 16, add `internal/hubreconcile_test#seedRetiredKey` to **Edit:**. State in the Intent that newStaleHub writes the full per-worktree file set first and calls seedRetiredKey after it. seedRetiredKey then commits batcher.yaml as the template with the retired key in place of orientation, plus the same one key whose value differs from the repo default. That way batcher.yaml still carries the retired key until a walk runs, and it survives the strip non-empty, so retiredKeyPresent's os.ReadFile finds it after the walk. The callers of seedRetiredKey that build on a marked pair, such as TestEnsure_PairRegisteredDuringTheWalkIsReconciledBeforeTheStamp, keep passing."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/16-sparse-migration.md
    summary: TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles is to assert the next walk strips the restored pair, but the hook never clears its merge record, so the next walk skips the pair again
    fix: "In card 16's Intent, say that the hook in TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles keeps the clear func that markMergeRecord returns. The test calls it after the restore assertions, then runs Ensure again and asserts the pair is stripped and committed with the sparse marker."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/16-sparse-migration.md
    summary: In card 16, runStaleBuildReconcile's second Ensure meets a stamp that is already current and never walks, so its a-second-walk-strips-nothing assertion passes without any walk running
    fix: "In card 16's Intent for TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree, say that runStaleBuildReconcile removes the stamp at geom.StampPath() before its second Ensure. It then asserts that the second walk lands no new prime, pair or board commit and leaves each worktree's per-worktree files and sparse marker byte-identical."
---

### [BLOCKING] Card 15 leaves the tests that expect the old clone output unplanned

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md`

**Issue:** Card 15 changes `ReconcileAll` in a hub so it never adds a key, and an absent worktree file stays absent.
The weft bare template that `buildBareTemplate` builds holds no config files (`internal/hubforge/hub.go`).
So after card 15, `CloneAndWire`'s `ReconcileAll` call writes nothing into the prime.
Its `CommitAnchoredPaths` call then has no paths, and the prime gets no `fabric clone: record module configs` commit.
The card says so itself: "its prime carrying only the per-worktree config files the records repository already held".
Existing integration tests still expect the old output, and neither card 15 nor any later card lists them:

- `internal/hubforge/hub_test.go`, `TestNewHub` step `ConfigMaterializedWithoutSeeding`, reads `configengine.ConfigFile(h.Location.AnchorPath(), "loom")` and calls `t.Fatalf` when the file is absent.
  Step `SeedConfigRedundantSeedDoesNotFatal` reads `configengine.ConfigFile(h.WeftBase, "loom")` and calls `t.Fatalf` the same way.
- `internal/fabriccli/cleanhub_integration_test.go`, `TestRunCLI_CleanHubScenario` step `CloneConfigCommit_WeftPrimeCleanAfterClone`, requires `git ls-files` in the prime to list every per-worktree module's config file.
  Step `CloneConfigCommit_OneCommitNotOnePerModule` requires the subject `fabric clone: record module configs` exactly once in the prime's log, and gets zero.
  Card 15 rewrites only the `CloneConfigCommit_MutationRecordShape` step, and card 16 only adds the marker assertions.
- `internal/orchcli/refusal_integration_test.go`, `TestOrchIntegration_Refusals` subtest `hub config reconcile`, builds its hub with `CopyHub` and `AddPair`.
  It reads `configengine.ConfigFile(h.PairCodeWorktree("orch-task"), "loom")` and calls `t.Fatalf` when the file is absent.

The plan's `## verify:` section runs `go test -tags integration ./...`, so card 15 cannot go green inside its own targets.
This is a gap in method, not one missed name: card 15 lists the hubforge doc and the clone's own scenario steps it rewrites.
It never derived which tests across the module read a per-worktree file that a hubforge clone used to write.
I checked the other tests that read a hubforge hub's per-worktree files.
The `webstercli` unreadable-config cases, `preflightshed` and `battencli` lifecycle seed their own file first and are unaffected.

### [MEDIUM] Card 16 leaves open the order of newStaleHub's new file set and seedRetiredKey

**Location:** `_lyx/plan/16-sparse-migration.md`

**Issue:** Focus 1 asks whether each worktree is unmarked and holds full per-worktree files when the walk under test starts.
Card 16's `newStaleHub` gives exactly that: it writes and commits a full set over the marked, file-less hub that cards 15 and 16 produce, and deletes the marker in the prime and in the pair.
The pair inherits the marker because `Topology.Add` forks from the local parent weft branch, so deleting it there is valid.

The card does not settle how the new writes compose with the `seedRetiredKey` calls that `newStaleHub` already makes, and either order breaks the tests:

- New set written after `seedRetiredKey`: `batcher.yaml` no longer carries `master_base`.
  `TestEnsure_CurrentStampLeavesConfigAlone` asserts `retiredKeyPresent(prime)` is true after a gated `Ensure`, so it fails.
  The not-reconciled checks in the other tests pass without testing anything.
- New set written before it: `seedRetiredKey` overwrites `batcher.yaml` with the template plus the retired key, which drops the one differing key.
  The walk then prunes the retired key and strips every remaining key as equal to the default, empties the file and deletes it.
  `retiredKeyPresent` calls `os.ReadFile` and `t.Fatalf`s on the missing file.
  That breaks `runStaleBuildReconcile`, the retry check of `TestEnsure_MidMergePairIsLeftAloneAndRetried` and the repair check of `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`.

`seedRetiredKey` is not among card 16's targets, so it cannot carry the differing key as the card stands.

### [LOW] The next-walk assertion after a restore cannot run, because the merge record stays set

**Location:** `_lyx/plan/16-sparse-migration.md`

**Issue:** Card 16 says `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` asserts "the next walk strips it".
The test's hook calls `markMergeRecord(t, h.PairRecordsSibling("pair-a"))` and drops the clear func it returns (`internal/hubreconcile/ensure_integration_test.go`).
The pair's `fabric-merge.json` therefore stays in place, and a next `Ensure` skips the pair as mid-merge again, the behaviour `TestEnsure_MidMergePairIsLeftAloneAndRetried` pins.
The no-marker-after-restore half holds: the snapshot is taken while `newStaleHub` has left the pair unmarked, so `restoreConfig` removes the marker the walk wrote.

### [LOW] The second-walk-strips-nothing assertion passes without a walk running

**Location:** `_lyx/plan/16-sparse-migration.md`

**Issue:** `runStaleBuildReconcile`'s second `Ensure` runs after the first one has written a stamp naming the running build.
The board defaults are present too, so card 15's absent-default check does not make the stamp stale.
No walk runs, and "a second walk strips nothing" passes without testing anything.
Card 16's `TestReconcileAll_MigratesToSparse` row "a marked worktree is left alone" covers the behaviour at configsync's surface.
The walk-level assertion the card states still needs a walk to run.

## Focus verdicts

- Focus 1, card 16's `Ensure` integration fixture: `newStaleHub` hands each test a prime and a pair that are unmarked and hold full per-worktree files when the walk starts.
  The asserted outcomes hold for the mid-merge retry and for no-marker-after-restore, except for the order defect in F2 and the uncleared merge record in F3.
- Focus 2, the clone output: cards 15 and 16 agree on the `internal/hubforge/doc.go` seeding contract.
  The defaults are committed on the board, the prime carries no per-worktree file, and the marker is committed by the clone's walk.
  They also agree on `CloneAndWire`'s mutation record: one `KindFileWritten` per seeded default, then one for the prime's marker, with no trailing weft commit.
  They agree on the `CloneConfigCommit_MutationRecordShape` step too.
  Two other steps of the same scenario, and `TestNewHub`, still expect the old output (F1).
- Focus 3, card order: no card before 15 relies on the clone's changed output.
  Until card 15, loom's strict load reads the prime file that the old clone still writes, and card 15 makes both changes together.
  Card 15's own walk tests run before any marker exists, and card 16's `newStaleHub` builds on the state that cards 15 and 16 have both produced.
  Cards 17 and 21 assume the sparse world: an absent pair worktree file in `TestDispatchLayers`, and no per-worktree change from `ReadConfigChanges`.
