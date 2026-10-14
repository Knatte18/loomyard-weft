---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/fabriccli/fabric.go:507
    summary: "lyx fabric add spawns its push through fabriccli's spawnPush (SpawnDetachedPush) and names the records worktree itself, not through fabricengine.SpawnPairPush as the summary, PATTERN-fabric-git and the decision record state"
    fix: In internal/fabriccli/spawn.go change the seam to `var spawnPairPush = fabricengine.SpawnPairPush` with the signature `func(*lyxcwd.Location) error`, and call `spawnPairPush(l)` at fabric.go:507 in place of the `codeWorktree, recordsWorktree` pair. The two KindPushSpawned entries keep `fabricengine.WorktreePath(l, slug)` and `fabricengine.RecordsWorktreePath(l, slug)` as their targets. Update `SetPairPushForTest` in internal/fabriccli/export_test.go and the TestMain replacement in internal/fabriccli/testmain_test.go to the `*lyxcwd.Location` signature, with the foreground body calling `fabricengine.PushPairAnchored(l, fabricengine.EnvSyncOptions(), fabricengine.LockWaitUnbounded)` as the battencli TestMain does. Update the spawn.go header comment to match.
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabriccli/fabric.go:507
    summary: no test covers the new add envelope (push_spawned entries per side, push_error on a failed spawn that still exits ok) or SpawnPairPush, although the acceptance criteria require them
    fix: In internal/fabriccli add a row to the existing add-verb test (extend it, do not add a top-level test) that swaps the seam through `SetPairPushForTest` for a recorder and asserts the envelope's mutations hold exactly two `push_spawned` entries targeting the code and records worktrees and no `push_error`. Add a second row where the seam returns an error and assert `push_error` carries it, the exit is ok and the pair still exists. In internal/fabricengine add a case to the existing spawn test file that sets `spawnDetachedPushFn` through its existing test seam and asserts `SpawnPairPush(l)` hands it `l.WorktreePath()` and `RecordsWorktree(l)`. If the new top-level tests push a package past its row, list cmd/lyx/testdata/test-budget.yaml under `Edit:`.
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/battencli/wire.go:826
    summary: the Worktree-Create push spawn, its warn-only failure path and the PushSeed spawn are never asserted; both battencli TestMain seams replace them with a foreground push or a constant error
    fix: Extend the existing TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating in internal/battencli/lifecycle_integration_test.go so it replaces `spawnPairPush` with a recorder for its duration (it is not parallel-safe, so name the process-global in the test comment, or put the recorder on the battenCLI value) and asserts one call with the new pair's location after the create row, and none on the re-entered complete pair. Add a row to the existing seamchild test in internal/battenshed asserting a PushSeed error leaves SeedChild Done, if that row is not already present. Keep the rows inside existing tests.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/fabricengine/destroy.go:444
    summary: the comment on ownedPairWeftBranch still places the replacement checkout at Add's step 12, which no longer exists
    fix: In internal/fabricengine/destroy.go:444 replace "at Add's step 12" with "while Add builds the pair".
  - id: F5
    severity: NIT
    class: design
    location: internal/fabricengine/add.go:341
    summary: addWarpBranch now holds only the creation token that rollbackAdd also receives separately as warpTok, so the wrapper and the extra parameter carry no information
    fix: In internal/fabricengine/add.go drop the `addWarpBranch` type and the `warpPush` variable, and make the rollbackAdd parameter that carries the branch token the plain `createdBranchToken` that the `warp.tok` uses today, updating the call sites at add.go:208 through 328 and the rollbackAdd doc comment. Skip this if the token passed as `warpTok` is a different value from `warpPush.tok`, and say so in the report.
---

### [MEDIUM] lyx fabric add bypasses SpawnPairPush

**Location:** internal/fabriccli/fabric.go:507
**Issue:** The summary says the callers of `Add`, `lyx fabric add` among them, spawn the push through `fabricengine.SpawnPairPush`, and `pattern/PATTERN-fabric-git.md` says "its callers push them through the detached `SpawnPairPush`".
Only batten's Worktree-Create row does.
`runAdd` calls `spawnPairPush`, which fabriccli defines as `spawnPush`, which wraps `SpawnDetachedPush`, and it builds both worktree paths itself.
The decision record asks for "one new engine entry" for both production callers.
The behavior is the same today, but the claim in a pattern file is false and the path-naming duplicates what `SpawnPairPush` exists to hide.

### [MEDIUM] The add envelope and SpawnPairPush are untested

**Location:** internal/fabriccli/fabric.go:507
**Issue:** The acceptance criterion "`lyx fabric add` spawns one detached push of the pair and reports it in its envelope" has no test.
A search of the tests finds no `push_spawned` or `push_error` assertion for add, and none that calls `SpawnPairPush`.
The `TestMain` replacement runs the push in the foreground, so a regression that drops the spawn or the entries would pass.
The deleted `pushed` assertions are replaced by nothing.
Under `PATTERN-test-economy` this is a behavior the diff adds with no coverage.

### [MEDIUM] Worktree-Create and Seed-Child push spawns have no assertion

**Location:** internal/battencli/wire.go:826
**Issue:** The new spawn after `Add` in the create row, with its two warn-only failure paths, and the `PushSeed` spawn replace behavior that was observable (a synchronous push reaching the remote).
Both `TestMain` seams in battencli make the calls foreground or constant failures, so no test pins that the create row spawns once per new pair and not on an already-complete pair, or that a failed spawn leaves the row Done.
The window-open part of the row is covered by the extended reconcile test.

### [LOW] Stale step number in destroy.go

**Location:** internal/fabricengine/destroy.go:444
**Issue:** The comment cites "Add's step 12", a step this change removed; the replacement is now step 11.

### [NIT] addWarpBranch is a one-field wrapper

**Location:** internal/fabricengine/add.go:341
**Issue:** With the push fields gone, `addWarpBranch` holds only `tok`.
`rollbackAdd` also takes `warpTok` as its own parameter.

## Rejected

- Scope against the board entry: the entry says only the window moves, and the diff also makes pair creation and the seed push detached.
  The decision record's parent decision of 2026-10-14 ("Scope: early window plus async pushes for pair creation and child seed", by: parent) supersedes the entry's "Scope" and "Out of scope" sections, and the diff makes exactly those three changes and no more.
  The decision sits under `## Decisions`, not under an `Added after Discussion` heading; I took the parent's own entry as the amendment, as the rubric ranks a parent entry above the board body.
- Leased deletion of the archived leftover as the last step: nothing runs after it, a failure there rolls the pair back as before, and a rolled-back `Add` never restores the deleted remote branch by design.
  `rollbackAdd` and the adoption paths no longer reference a pushed branch; a search finds no leftover reader of `Pushed`, `pushSeam`, `warpOriginState` or `branchDirtinessPushedByThisCall`.
- Once-marker: `OpenTerminalOnce` and the Run-Shed row use `battenrecipe.NameRunShed` and `BattenDir(location, slug)`, the same marker `ClearTerminalOpened` resets when a fresh batten status is seeded, and the open runs only after `Add` and `reconcileTaskPair` succeed and only under `--open-terminal`.
- `lyx fabric add` records `push_spawned` even when `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` made the spawn a no-op; `lyx fabric sync` does the same, so it matches the existing convention.
