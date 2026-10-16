---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md:33
    summary: The verify section never runs ./tools/sandbox, the package whose go:embed inputs and TestRunSuite_PerSuiteSpec read the three suite files card 6 edits
    fix: In 00-overview.md's verify block, add ./tools/sandbox to the untagged line (lyx gate test ./internal/agentname ./internal/reedengine ./internal/reedcli ./internal/shuttleengine/claudeengine ./tools/sandbox ./cmd/lyx) and to the --tags integration line in the same position; leave the go build and --tags tmux lines unchanged. Checked with validate-plan --overlay together with F4's fix and found no new finding.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-watchdog-stop-and-recheck.md:58
    summary: The nil-return branch's unless cmd.Context() is done guard on the exit re-check has no test row that reaches it; the cancel row reaches the ctx.Err() branch instead
    fix: In card 4's TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping row list, add a row stop with an idle listing and a cancelled context (IdleCycles 1000000) in which the lister script cancels the command's context during the stop-woken cycle's call, answers idle on that call and affirmative on every later call. The command returns nil from the loop, and the row asserts no respawn and no lister call after the cancelling one. Add a sentence to the Intent naming this row as the test of the nil-branch guard.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-down-waits-for-watchdog.md:31
    summary: downCmd's c.hubPath non-empty guard and its empty-hubPath test row defend a state no reed CLI input reaches, because PersistentPreRunE always sets c.hubPath from location.HubPath
    fix: In card 5's Intent, drop the and c.hubPath is non-empty (hub mode) condition so the wait runs whenever EndedServer is true. Add a sentence saying every reed CLI engine is built from hubgeom.ReedGeometry with c.hubPath set, so the standalone half of the acceptance criteria is carried by card 3's no-path-told row. Delete the empty hubPath row from TestDownCmd_WaitsForTheWatchdogOnlyWhenItEndedTheServer's row list.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/03-down-writes-discover-stop.md:7
    summary: TestTouchDiscoverSignal_TruncatesAPendingStop is a new top-level test of two unexported helpers for one assertion that fits the last-session row of card 3's own TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer
    fix: In card 3, delete the Create bullet for plan:internal/reedengine#TestTouchDiscoverSignal_TruncatesAPendingStop and the paragraph that describes it. Extend the last-session row of TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer so that, after asserting the marker, it calls touchDiscoverSignal and asserts the file exists and is empty, so a cold boot after a stop and before the daemon consumes it turns the stop back into a discover. Checked with validate-plan --overlay together with F1's fix and found no new finding.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/04-watchdog-stop-and-recheck.md:77
    summary: The rewritten exit-path table keeps the existing rule that a row with no expected trace leaves the hub logs directory empty, but the new looping rows log Info lines such as watchdog daemon starting, so they fail that check
    fix: In card 4's TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping paragraph, say the empty-logs-directory assertion is limited to the lock already held row. Every other row asserts its own trace lines instead, for example the stop-with-idle-listing row's exit-on-stop Info line, and the no-successor rows assert that no successor-spawned line appears.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/04-watchdog-stop-and-recheck.md:82
    summary: The stop-with-affirmative-listing row checks that the command is still running after one more lister call, but the cycle's stop-or-continue decision follows that call, so a wrong exit can still pass the check
    fix: In card 4's stop with an affirmative listing row, replace and one more lister call with and two more lister calls. The second call proves that the stop-woken cycle finished and the loop started another cycle.
---

### [MEDIUM] Verify leaves the sandbox suites' own package unrun

**Location:** `_lyx/plan/00-overview.md:33`
**Issue:** Card 6 edits `tools/sandbox/SANDBOX-REED-SUITE.md`, `SANDBOX-SHUTTLE-SUITE.md` and `SANDBOX-CREEL-SUITE.md`.
`tools/sandbox/suite.go` embeds each of them with `//go:embed`, so they are build inputs of the `tools/sandbox` package, which has no `go.mod` of its own and is in the main module.
`TestRunSuite_PerSuiteSpec` in `tools/sandbox/suite_test.go` iterates `reedSuite`, `shuttleSuite` and `creelSuite` and asserts that each rendered scheme contains the embedded suite body.
The plan's `## verify:` runs `./cmd/lyx` (whose `sandbox_coverage_test.go` globs the files) but never `./tools/sandbox`, so the targeted package's own tests go unrun.

### [MEDIUM] The re-check's cancelled-context guard on the nil branch is unreached

**Location:** `_lyx/plan/04-watchdog-stop-and-recheck.md:58`
**Issue:** Card 4 says that after a nil loop return, `watchdogCmd` re-lists the hub's sessions "unless `cmd.Context()` is done".
That guard is how D3a's "A cancelled context spawns nothing" holds when the loop leaves by a stop, the idle-cycle rule or an idle replaced retire while the context is cancelled.
The only cancelling row, "stop with an affirmative listing", cancels while the loop waits in its `select`, so `runWatchdogLoop` returns `ctx.Err()` and the command takes the "any other error" branch.
No row reaches the nil branch with a done context, so the prescribed guard has no named test.
The `list` seam this card adds makes that row hermetic.

### [LOW] downCmd's hub-mode guard defends an unreachable state

**Location:** `_lyx/plan/05-down-waits-for-watchdog.md:31`
**Issue:** `internal/reedcli/cli.go` builds `c.eng` only in the parent's `PersistentPreRunE`, from `hubgeom.ReedGeometry(location)`, and in the same step sets `c.hubPath = location.HubPath`.
`lyxcwd`'s `resolveCore` always sets `HubPath` to `filepath.Dir(workTreeRoot)`, so it is never empty, and reedcli has no standalone or `--target-dir` path.
The `c.hubPath` non-empty condition and the "empty `hubPath` (no hub mode)" row therefore pin a state only a hand-built `reedCLI` can reach.
D3's "in hub mode" holds for every real `down` without the guard, and card 3's no-path-told row already carries the "Standalone `down` writes no signal" criterion at the engine, the only layer where a geometry can tell no path.

### [LOW] Truncation test is a separate top-level test for one row's worth of assertion

**Location:** `_lyx/plan/03-down-writes-discover-stop.md:7`
**Issue:** `TestTouchDiscoverSignal_TruncatesAPendingStop` calls the unexported `writeDiscoverStop` and `touchDiscoverSignal` and checks a single fact: the touch empties a pending stop.
Card 3 already adds the table-driven `TestDown_WritesTheDiscoverStopOnlyWhenItEndsTheServer`, whose last-session row leaves exactly that pending stop on disk.
`PATTERN-test-economy` turns such a case into a row or assertion of the existing test of that behavior instead of a new test function.
The public-surface test of the cold boot's touch, `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`, is tmux-tier, so folding into card 3's hermetic table keeps the case untagged.
`internal/reedengine` stays within its test-budget row either way (204 counted today against 270).

### [LOW] New looping rows contradict the table's empty-logs assertion

**Location:** `_lyx/plan/04-watchdog-stop-and-recheck.md:77`
**Issue:** Today `TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` asserts that the hub logs directory holds no files whenever a row's `wantTrace` is empty.
Only the lock-held row runs no loop.
The binary-replaced row already shows that the loop's Info lines reach the durable sink at the default verbosity: it finds "reed: watchdog retired" there.
Card 4 adds rows that run the loop but name no trace expectation (the idle-cycle exits, the stops), and `runWatchdogLoop` logs "reed: watchdog daemon starting" at Info on every start.
Implemented as the card reads, with "lock already held: unchanged" and no word on the trace rule, those rows fail the empty-logs check.

### [LOW] The stop-with-affirmative row can pass on a wrong exit

**Location:** `_lyx/plan/04-watchdog-stop-and-recheck.md:82`
**Issue:** The stop flag stays pending from consumption until `afterCycle`, so the first lister call after the file disappears belongs to the stop-woken cycle.
Card 4's cycle order makes the replaced check and the stop-or-idle decision run after that listing returns.
A test that sees one more lister call and then checks "still running" can therefore check before the decision runs, and a loop that wrongly exits on a stop with an affirmative listing would pass.
A second call after consumption can only come from a later cycle, which proves the stop-woken cycle continued.

## Rejected

- Card 3's `Engine.ensureServerGone` seam: licensed. The acceptance criterion "A `down` whose server teardown does not confirm gone still writes the stop marker, returns its error" needs a test. `ensureServerGoneLocked` fails only when `reapSocketProcesses` cannot kill a socket holder, which no hermetic fixture stages. The seam is nil-by-default and production never sets it.
- Card 5's `strandOps.Down` widening: licensed by D3's injectable bound and poll. `*reedengine.Engine` already has `Down() (DownResult, error)`, and `fakeStrandOps` is the existing seam in `removedetach_test.go`.
- Card order across cards 4 and 5: card 4's `DownStopsTheDaemonAndReleasesLock` step uses only card 3's `Engine.Down` stop. Card 5 depends on card 4's step and card 3's `EndedServer`, and no card uses a symbol only a later card defines.
- Fidelity: D1 and the engine half of D3 are in card 3, D2, D3a and D4 in card 4, D3's verb half in card 5, D5 and D7 in card 2, D6 in cards 1 and 2, and D8 in card 6. Every acceptance criterion maps to a named row or step. Card 6 points its creel bullet at the reed suite's bullet rather than at the decision record's Scope section, which is fine: the reed bullet states the exclusion in a file that lasts.
- Reachability in card 4: the read error comes from `os.ReadFile` on a directory, and the removal error from a read-only parent, skipped on Windows and as root, where unlink would succeed. The replaced idle retire is reached through `watchdogExeStat` answering the second binary after the baseline stat.
- Reachability in card 5: the probe error comes from `flock` opening a lock under an absent scratch directory, and the timeout Warn from a scripted always-held probe over `wait/poll + 1` probes.
- Attack surface: card 2 bounds the prepend to `LYX_BIN`'s parent directory. Card 4 bounds the stop to idle listings and the one-second `StopFloor`, and bounds a transient list error to one successor spawn. Card 3's marker is honoured only through card 4's bound.
- Glyph hygiene: every re-signed member (`consumeDiscoverSignal`, `discoveryWake.onSignal`, `runWatchdogLoop`, `runWatchdogCmdInBackground`) carries the arrow. `Engine.Down`, `reedCLI.downCmd`, `afterCycle`, `next` and `startReapLoop` keep their signatures and carry none, and the `strandOps` interface takes none.
- Test budget: `internal/reedcli` has 39 top-level tests today against a row of 40, so card 5's one new test fits without editing `test-budget.yaml`.
- No card plans an assertion on a stencil's static prose.
