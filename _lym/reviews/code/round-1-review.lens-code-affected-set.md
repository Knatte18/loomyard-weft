```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines they touch, and whether the diff leaves any of it wrong.

**Per change, the set found:**
- **Recording board sync installed once in `TestMain`** (`testmain_integration_test.go:360`).
  - `SetSpawnBoardSyncForTest`'s doc (`export_test.go:17-19`) still holds, since `TestMain` calls it before any test runs: covered.
  - Production `walk.go:216-218,296`: unchanged.
  - The package's internal tests (`carry_test.go`, `stamp_test.go`, `seam_enforcement_test.go`) don't call the seam: covered.
  - The real `boardengine.SpawnSync` (`internal/boardengine/spawn.go:24-28`) is no longer reached by any test of the binary. Finding 1.
- **Recording notifier installed once in `TestMain`** (`testmain_integration_test.go:361`).
  - Production installer `internal/orchcli/reconcilenotifier.go:27`.
  - `carry_test.go`'s walks now mail into the recorder where they used to find no notifier. None of them asserts on the notifier: covered.
  - The scripted answers (`mailAnswer`) stand in for orchcli's notifier. Finding 2.
- **Removed tests and helpers** (`recordBoardSyncs`, `recordNotices`, `TestEnsure_BoardWithNoRemote…`, `TestEnsure_PairRegistered…`, the three reinterpretation tests, `TestEnsure_UnpushedHubWideCommitOverMovedUpstream…`).
  - References survive only in dated research and measurement files under `docs/research/` (`fabric-batten.md:82-83`, `tests.tsv`, the `candidates.tsv`/redundancy JSON snapshots). Those are measurement reports, so they are exempt.
  - `docs/benchmarks/test-redundancy.md` has no hubreconcile section.
  - `fabricengine`'s `recordNotices` is a different symbol.
  - Covered.
- **Serial-test comments** (PATTERN-test-speed, "Tests run in parallel").
  - Every remaining serial test or row names its state: logger at `ensure_integration_test.go:625,751,1118`, seam at `:1157-1159`, registry and notifier at `reinterpret_integration_test.go:63-65`.
  - The `ensure_integration_test.go:4` file header matches the code.
  - Covered.
- **Rewritten file headers** (`ensure_integration_test.go:1-4`, `reinterpret_integration_test.go:1-3`, `testmain_integration_test.go:1-4`): accurate against the code. Covered.
- **Commit messages** over `2326ef4da..HEAD`:
  - No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer.
  - The body of f2ebdb2c1 misstates which tests run alone. Finding 3.
  - Removed assertions carry no trailer. Finding 4.
- **Declared PATTERN changes:** the directive declares none, so nothing to land.

**Findings:**

1. **BLOCKING, design.** `internal/hubreconcile/testmain_integration_test.go:372-378` (`boardSyncRecorder.record`). The recorder replaces `boardengine.SpawnSync`, another repository module's function, and never calls it.
   - PATTERN-test-stand-ins forbids a stand-in for a repository module. It allows only "a wrapper that records or counts calls and passes each one to the real module" (`pattern/PATTERN-test-stand-ins.md`, "What is not a stand-in").
   - Before this diff, the parallel tests reached the real `SpawnSync`. Now no test in the binary does.
   - The real call is free under test: `SpawnSync` returns nil when `testing.Testing()` holds (`internal/boardengine/spawn.go:24-28`). So the fix is to count the call, then return `boardengine.SpawnSync(boardPath)`.
   - The base's `recordBoardSyncs` had the same shape, but this diff writes the new code and widens it to every test.
2. **MEDIUM, design.** `internal/hubreconcile/testmain_integration_test.go:401-408` (`mailRecorder.send`, `mailAnswer`). The notifier recorder answers "posted", "not posted" or an error by hand. That scripts the outcomes of orchcli's notifier (`internal/orchcli/reconcilenotifier.go:27`), which is a repository module, so it is a stand-in under PATTERN-test-stand-ins.
   - The base already did this (`recordNoticesAnswering`), and `docs/research/no-fake-other-modules/fabric-batten.md:82` records the real notifier as the target.
   - Driving the real notifier is beyond a test-time cut, so I rate it MEDIUM rather than BLOCKING. The handler decides.
3. **LOW, consistency.** Commit f2ebdb2c1's message: "only the logger-capturing tests and the no-notifier row still run alone".
   - The `FailingSeam` row also runs alone, before its siblings (`ensure_integration_test.go:1157-1166`).
   - The message leaves that row out, although the change description and the test comment both name it.
4. **MEDIUM, design.** `internal/hubreconcile/ensure_integration_test.go:626-677` (`TestEnsureOrWarn_WarnsWithTheWayForward`). The diff removes the after-repair assertions "EnsureOrWarn warned after the repair" and "stamp absent after the repaired reconcile", and no commit carries a `Loosened-Check:` trailer for them (base lines removed at diff part 1, 335-348).
   - The stamp half is still held: the "unparseable pair config" repair row (`:526`, `:613-619`) checks it.
   - No test asserts that `EnsureOrWarn` logs nothing once `Ensure` succeeds. The only `EnsureOrWarn` test is this one (grep found no other test file calling it), and its code is at `hubreconcile.go:67-71`.
   - The change description's claim that "another row already covers the retry" does not hold for the no-warning half.

**Tool calls:**
- Grep for references to the removed test names and helpers, repo-wide: the manifest did not cover docs or fixtures that name them.
- Read `docs/benchmarks/test-redundancy.md` (grep), the `PATTERN-test-stand-ins.md` background and `boardengine.SpawnSync`: the stand-in rule and what the recorder replaces were not in context.
- Grep the package's other test files for the notifier and sync seams, and for production `InstallNotifier` callers: needed to know who fills the seam in production and whether other tests in the binary are affected.
