```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the pinning test and applied test economy. I read test names and a few bodies, not every test body.

**Behaviour → pinning test**
- **Worktree-gone exit (D1):** `TestRun_WorktreeGoneExits` in `internal/dobbyengine/loop_worktreegone_test.go:103` covers these cases:
  - the tidy is stopped through the prime runner, and the line lands in the prime's log
  - a failed stop names `lym reed remove`
  - no tidy in flight
  - the kept tidy comes from the last tick that ended with the git link present
  - the exit comes at the top of the next iteration, with no second tick
  - an unreadable record keeps the earlier tidy
  - an undecodable record reads as no tidy
  - a prime whose git link is missing keeps ticking
  - The focus's "woken by a file event" case is probably the "top of the next iteration" row, but I did not confirm it.
- **Re-exec test (D2):** `TestReadsAsReexec` (`loop_test.go:580-584`) pins the pid and ticks matrix.
  - `Run` start rows in `loop_test.go` cover the 10 s wait versus `lock.ProbeGrace`, the no-ticks record and the full-pass rule.
  - They also cover the ticks rewrite and the second state read on a refused re-exec.
  - The tick refill of a record with no pid is asserted at about `loop_test.go:324-350`.
- **Exit lines and notices (D3):** `loop_test.go` has rows for these:
  - a refused fresh start
  - an owned-elsewhere exit, with and without a slug
  - a failed exec
  - `Locked` errors
  - an unreadable state record
  - `TestRunDobbyWith_TracesEachExitBeforeRun` (`internal/beamcli/dobbytrace_integration_test.go`) covers wiring failure and `placeDobby` refusal.
  - I did not see a `Run` row that pins "fatal error after a re-exec posts one notice" separately from the `Locked` row.
- **`internal/selfexec` callbacks:** `selfexec_test.go` was edited, but I did not open it.
- **Log rotation and `dobby.out` (D5):**
  - `TestAppendLog` pins the lock-holder-only rotation.
  - `TestRotateOutBeforeSpawn` pins Windows with a free lock, Windows with a held lock, Linux, and a file under the bound.
  - `TestSpawnDetached_SendsOutputToOutPath` pins the spawn.
  - `TestDobby_ReexecsOntoAReplacedBinary` (tier1) pins the fd reopen at lines 137-204, in `internal/beamcli/dobbyreexec_integration_test.go`.
- **Changes-only log and heartbeat (D6):** `TestLog_ChangesOnly`, `TestRun_Heartbeat`, and the `loop_test.go:677-700` assertions for tick failing/succeeding again and owner unreadable/reads again.
- **Machine on history entries (D7):** `TestStep_StampsTheToldMachine` covers the producer-call, skipped and reroute entries and an untold shed. `TestBuildShed_TellsTheMachine` covers batten telling the name and refusing on a name error.
- **Landing filter and machine flag (D8):** these tests cover it:
  - `TestDobbyLandings_CountsOnlyThisMachine`
  - `TestMachineLandings`
  - three rows in `tidy_test.go` (emptied once, a tidy in flight settles first, baselined and flagged)
- **Adopt gate seed (D9):** `TestSeedAdoptGate`, `TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize`, and `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers`.
  - I did not see the `TestMigrateConfig` "no longer seeds adopt" row that the first Added-after-Discussion entry asks for.
  - Only `TestSeedAdoptGate` for `SeedAdoptGate` and the hub-reconcile test were added.
  - `config_test.go` has +67 lines, so the row may be inside an existing function.
- **In-process gate with observer:** the new subtest in `gatecli/scopes_integration_test.go` covers a started tier, a carried tier and a blocked tier.
- **Check verb:** `TestRunCheck`, `TestCheckCmd_FromABeamExitsZeroWhileOneRuns`, `TestEnsureCheck` and `TestCheckAlive`.
- **Beam status for the check:** `TestCheckView` (running, green, failed, dead) plus `TestCheckAlive` for a pid alive with differing ticks. `status_integration_test.go` adds only lines to the existing test.
- **Test placement:** the binary-replacing test is tier1 (`//go:build tier1` at `dobbyreexec_integration_test.go:1`), as D-constraints require. `loop_test.go` and `loop_worktreegone_test.go` are tier0.

**Findings (one, plus one unconfirmed)**
- LOW, `consistency`, `internal/gateslot/config_test.go`: I found no test, among the added function names and the 67 added lines, named as the `TestMigrateConfig` row that pins `MigrateConfig` no longer seeding `adopt`. The first Added-after-Discussion entry requires that row.
  - Fix: confirm a `TestMigrateConfig` row asserts that a document tuning `finalize` with no `adopt` comes back from `MigrateConfig` without `adopt`.
  - If it is absent, add it.
  - This is unconfirmed: the row may be an extra case inside an existing function that my name filter cannot show.
- Nothing found for duplicate tests or per-helper tests. `TestMachineLandings` (the seam wrapper) and `TestDobbyLandings_CountsOnlyThisMachine` (the filter) cover different behaviours.

**Tool calls (2 of 6)**
1. `git diff -U0` filtered for test function names and table rows across all changed `_test.go` files. My context held the production diff but not the tests.
2. `grep` of test names and build tags in `loop_worktreegone_test.go`, `loop_test.go` and `passes_test.go`/`tidy_test.go`/`start_test.go` diffs. My context lacked the dobbyengine subtests and tier tags.
3. `grep`/`sed` on the dobbyengine, beamcli and adoptcli tests and the `selfexec`/`gatecli` test diffs. My context lacked the episode-log assertions, the reopen assertions and the landings wrapper test.
