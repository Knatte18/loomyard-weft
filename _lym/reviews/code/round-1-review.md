---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/hubreconcile/testmain_integration_test.go:373
    summary: The binary-wide board-sync recorder stands in for boardengine.SpawnSync and never calls it, so every test of the binary, including the ones that reached the real sync before the diff, now runs against a stand-in PATTERN-test-stand-ins forbids.
    fix: "In boardSyncRecorder.record, after incrementing r.calls[boardPath] under the lock, return boardengine.SpawnSync(boardPath) instead of nil, and import github.com/Knatte18/loomyard/internal/boardengine. Change its doc comment from \"counts one sync of boardPath and starts nothing\" to say it counts the sync and passes it to the real detached sync. SpawnSync returns nil without spawning in a test binary, so no assertion changes."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/hubreconcile/ensure_integration_test.go:637
    summary: The diff removes the broken-config row's after-repair assertions that EnsureOrWarn logs no warning once the reconcile succeeds and writes the stamp, with no Loosened-Check trailer. No other test asserts that EnsureOrWarn stays quiet on success.
    fix: "Restore the repair to the \"broken config is warned about\" row of TestEnsureOrWarn_WarnsWithTheWayForward. setup returns a third value, a repair func, which is commitTemplate(t, h.PairRecordsSibling(\"pair-a\"), \"loom\") for this row and nil for the held-lock row. After the existing assertions, when repair is non-nil, call repair(), logs.Reset() and hubreconcile.EnsureOrWarn(geom, opts) again. Then assert that logs.String() does not contain wayForwardText and that stampKey reports a stamp. Rename the row back to \"broken config is warned about, then reconciled once fixed\", and correct the summary's \"Retry walks removed\" bullet so it no longer lists the EnsureOrWarn row."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: lens:code-affected-set
    location: internal/hubreconcile/testmain_integration_test.go:402
    summary: The notifier recorder scripts orchcli's posted, unposted and error answers. The base already had this inventoried standing fake, but the diff installs it for every test of the binary, so walks that had no notifier now mail into a scripted one.
    fix: "Leave the code in place and defer this to the run's parent. The parent rules whether the standing notifier fake, which docs/research/no-fake-other-modules/fabric-batten.md lists for a later sweep, may serve every test of the binary until that sweep replaces it with the production notifier. A fixer cannot drive the real notifier inside this test-only cut."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-scope
    location: _lym/landing/summary.md:19
    summary: "The net test-line figure, 394 added and 450 removed, so 56 fewer, counts commit f2ebdb2c1 alone. Over the range, git diff --stat 2326ef4da..HEAD gives 395 insertions and 450 deletions, so 55 fewer."
    fix: "In _lym/landing/summary.md, change the line to \"Net test-line change: 395 added, 450 removed, so 55 fewer lines, all in `internal/hubreconcile`.\" Re-derive it from git diff --stat over the range after any fix commit."
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/hubreconcile/ensure_integration_test.go:1163
    summary: "The body of commit f2ebdb2c1 says only the logger-capturing tests and the no-notifier row still run alone, but the FailingSeam row of TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree also runs alone before its parallel siblings."
    fix: "When the fix commits are made, reword the body of f2ebdb2c1 to name the refusing-seam row as running alone next to the logger-capturing tests and the no-notifier row. If history is not rewritten, put that correction in the fix commit's message."
---

### [BLOCKING] The board-sync recorder replaces boardengine.SpawnSync instead of wrapping it

**Location:** `internal/hubreconcile/testmain_integration_test.go:373`

**Issue:** `TestMain` fills `SetSpawnBoardSyncForTest` with `boardSyncRecorder.record`.
That function counts the call and returns nil without calling `boardengine.SpawnSync`, which fills the seam in production (`walk.go:218`).
Before the diff, only the tests that called `recordBoardSyncs` swapped the sync.
The rest, among them `TestEnsure_MidMergePairIsLeftAloneAndRetried`, `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`, `TestEnsure_ConcurrentCallsLandOneBoardCommit` and `TestEnsureOrWarn_WarnsWithTheWayForward`, reached the real `SpawnSync`.
Now no test in the binary does.
PATTERN-test-stand-ins forbids a stand-in for another repository module, and "No change adds a stand-in this rule forbids."
It names the allowed shape: "A wrapper that records or counts calls and passes each one to the real module."
That shape costs nothing here, since `SpawnSync` returns nil without spawning when `testing.Testing()` holds (`internal/boardengine/spawn.go:24-28`).
The stand-in inventory (`docs/research/no-fake-other-modules/fabric-batten.md:83`) also names a recording wrapper as the target for this seam.

### [MEDIUM] EnsureOrWarn's quiet success lost its only assertion

**Location:** `internal/hubreconcile/ensure_integration_test.go:637`

**Issue:** The base row "broken config is warned about, then reconciled once fixed" repaired the config, called `EnsureOrWarn` again, and asserted two things: no way-forward warning, and a stamp.
The diff drops both checks, and no commit carries a `Loosened-Check:` trailer.
The stamp half is still held, by `Ensure`'s retry in the "unparseable pair config" row (`:526`, `:613-619`).
The no-warning half is held nowhere: a grep finds no other test that calls `EnsureOrWarn`.
So an `EnsureOrWarn` (`hubreconcile.go:67-71`) that warned on every call would pass every test.
The change description's claim that "another row already covers the retry" does not hold for this row.

### [MEDIUM] The scripted notifier now serves every test of the binary

**Location:** `internal/hubreconcile/testmain_integration_test.go:402`

**Issue:** `mailRecorder.send` answers each mail as posted, unposted or failed, as the test of that board set it.
In production, only `orchcli`'s notifier fills `InstallNotifier` (`internal/orchcli/reconcilenotifier.go:27`), so the recorder scripts that module's outcomes.
The base had the same scripted answers in `recordNoticesAnswering`, a standing fake that the inventory lists for replacement by "the installed production notifier over a real box".
The diff installs it for the whole binary, so the walks of tests that installed no notifier, which took the nil-notifier branch before, now mail into the scripted one.
Unlike F1, no cheap pass-through exists, since the production notifier needs a real box.
Whether the standing-fakes clause covers this widening until the sweep is the parent's ruling, not the fixer's.

### [LOW] The net test-line figure is stale

**Location:** `_lym/landing/summary.md:19`

**Issue:** The board entry asks for the net test-line change in the change description.
The figure given matches `git show --stat f2ebdb2c1`, 394 insertions and 450 deletions.
The range's diff, with the rewrap commit `f076676a2` (10 insertions, 9 deletions), gives 395 insertions and 450 deletions.

### [NIT] Commit message misses one row that runs alone

**Location:** `internal/hubreconcile/ensure_integration_test.go:1163`

**Issue:** The body of commit `f2ebdb2c1` says "only the logger-capturing tests and the no-notifier row still run alone."
The `FailingSeam` subtest also runs serially, before the heddle seam is installed for its parallel siblings.
The test's doc comment and the change description both say so.

## Rejected

- code-tests NIT, the non-mapping template base row lost its retry: that retry walks the same hub-wide path as the retry of the "unparseable hub-wide config" row (`:550-553`), which still stamps a clean board, so no behaviour loses its test.
- code-tests residual, whether the removed "a hub file with no value to carry" row was the only cover for the absent-file branch: `fabriccli` seeds absent hub files sparse on the board (`internal/fabriccli/fabric.go:671`). So the forged board holds a `slack.yaml`, and that row reached `Rewrite` with unchanged bytes, as the remaining value-5 row does.
- Handler, a serial test now installs the Bolt replayer before every parallel test runs: `RegisterBoltReplayer` is built for parallel sharing (`internal/fabricengine/boltreplay.go:61-63`), and production always registers it (`cmd/lym/main.go:174`). The parallel tests now run closer to production, and the tier1 run under `-race` passed with no race.
- Handler, the "Names of the tooling" bullet against the change description: `lym gate test` and its tiers are this repository's own subject, and the description names no board entry, Sketch, decision record or parent record.
- Handler, the timing claims: the description gives before and after figures for each package. A rerun here of `lym gate test --tier 1 ./internal/hubreconcile -- -count=1` passed with a wall time of 11.5 s, build included, in line with the claimed cut from 29.5 s. The same run under `-race` passed with no data race reported.
