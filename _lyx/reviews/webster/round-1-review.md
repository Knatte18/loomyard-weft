---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/battenshed/innerrun.go:207
    summary: New and changed Go comments break PATTERN-comment-line-breaks with fixed-column wraps and several sentences on one line
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/reviewstatus_test.go:273
    summary: New tests that touch no global state omit t.Parallel, and two new tests that swap the logger output name that state nowhere, against PATTERN-test-speed
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/websterengine/verifygate.go:362
    summary: verifyGateStuckReason has no timeout case, so a gate that exhausts its attempts on timeouts reports the failing identities as none named
  - id: F4
    severity: LOW
    class: design
    location: internal/battenshed/innerrun.go:446
    summary: The revive once-marker survives the history-length episode boundary that callHalted uses for its halt Warn, so a re-halt with a dead driver is not revived by the same process
  - id: F5
    severity: LOW
    class: consistency
    location: internal/reedengine/strand_test.go:530
    summary: TestStrandNamed repeats rows TestAddStrandUnless_NamedStrandSkips already covers, and its testtiming keep reason is now false
  - id: F6
    severity: LOW
    class: consistency
    location: docs/overview.md:373
    summary: The changed loom bullet in the module list keeps two sentences on one line
---

The review range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, no `_lyx/loom/rework` round).
`go build ./...`, `go vet ./...`, `go test ./...`, `go test -tags integration` over the touched packages and `go test -tags tmux ./internal/reedengine/... ./internal/reedcli/...` all pass, and `GOOS=windows go vet ./internal/verifyrun/` is clean.
The plan carries no `Rename` or `Delete` groups, so the per-type mechanical checks do not apply to any card.

The focus questions settle as follows, none yielding a finding beyond those below:

- Cards 7–9: `procgroup_unix.go` sets `Setpgid` and `Cancel` sends `SIGKILL` to the negative pid, and `TestRun_ExpiryKillsBackgroundedDescendant` proves a backgrounded `sleep` dies.
  `verifytree.Verify` turns a deadline into `StatusFailed` with `TimedOut` and returns an error only for the caller's own cancel.
  All four production call sites (`loomshed`, `websterengine`, `landingshed`, `webstercli`) read it as a failed verify, none as an error.
- Cards 1, 11, 13: every `SetWaitMark` call and every marker write or removal in `waitmark.go` and `landingshed/verifygate.go` is logged at Warn and changes no verdict.
  Both markers `lyx loom status` reads carry a pid, and `ReadMarker` and `ReadWaitMarker` treat a dead pid as absent.
- Cards 14 and 17: `lyx loom resume` and batten's revive read reed's `Directory` through `driverDirectory` and `driverStrandFrom`, never the seed's driver.
  Resume wakes only a parked driver and refuses every other state with a way forward, each refusal having a `refusal-spec.md` row and a reaching test.
  Batten revives once per episode through `ReviveStrands` and never resumes the child.
- Card 16: `spawnConfirmed` is true only for this process's pid, and a live or retiring driver strand or a held run lock adopts the child instead of re-spawning it.
- Cards 3, 5, 6: `strandNamed` ignores liveness and anchor, `planResumeLaunches` drops a non-live strand only when its done-when list is non-empty and fully present, and `Runner.start` sets `DoneWhen` only for an ungated run.

### [BLOCKING] Changed Go comments break PATTERN-comment-line-breaks

**Location:** `internal/battenshed/innerrun.go:205-213` and `:633`, `internal/battenshed/innerrun_test.go:406-409`, `internal/reedengine/lifecycle.go:638-639`, `internal/reedengine/windowsize.go:138-145`, `internal/reedcli/resume.go:1-3`, `internal/tokenvocab/doc.go:6-9`, `internal/tokenvocab/tokenvocab_test.go:79-80`, `internal/reedengine/waitmark_integration_test.go:3-5`.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line with no column limit for new and changed comments.
The diff rewrote lines inside fixed-column-wrapped paragraphs and left them wrapped mid-sentence, and several changed lines carry two sentences (`... holding this process's pid. A write failure is logged`, `... a second driver. A halted or done child is never re-spawned ...`, `... with no driver behind it. The confirmation is a marker ...`, `... before the first test. A child bootstrap seeds ...`).
`waitmark_integration_test.go` is a new file whose header is wrapped mid-sentence throughout.
**Fix:** Re-break every changed or new comment paragraph at sentence boundaries, and at independent-clause boundaries inside a long sentence, touching no untouched line.

### [BLOCKING] New tests omit t.Parallel without naming global state

**Location:** the new tests `TestFormatElapsed`, `TestShuttleWaiting`, `TestResumeVerb` and its two siblings in `internal/loomcli`, `TestDriverStrandFrom` in `internal/battencli/wire_test.go`, `TestSetWaitMark` in `internal/reedengine/io_test.go`, `TestStrandNamed`, three new `TestInnerRun_*` tests in `internal/battenshed`, and `TestGate_WaitMarkAroundEntry`, `TestShellWaitMark`, `TestWait_ClearsAStaleWaitMarkOnEntry`, `TestRunner_StartGated_CarriesNoDoneWhen` and `TestReadWaitMarker_AbsentRunDirectoryRootReadsAsNone` in `internal/shuttleengine`.
**Issue:** `PATTERN-test-speed` says a test calls `t.Parallel` unless it touches process-global state, and a comment names that state.
These tests run over injected fakes, per-test temp directories and pure functions, and touch no process-global state, yet none calls `t.Parallel`.
Separately, the new `TestInnerRun_HaltedRevivesADeadDriverOncePerEpisode` and `TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn` replace the process-global logger output, which forbids `t.Parallel`, and nothing at the tests or atop their file names that state.
**Fix:** Add `t.Parallel()` to the tests that touch none, and a sentence atop `innerrun_halted_test.go` naming the logger output swap for the two that do.

### [MEDIUM] A timed-out verify gate that exhausts its attempts reports no identities

**Location:** `internal/websterengine/verifygate.go:362` (`verifyGateStuckReason`).
**Issue:** `failTimedOut` writes a report with `TimedOut`, `LogPath` and `LogTail` and no `Failures` or `Dirty`.
When the attempts run out, `verifyGateStuckReason` finds neither list and renders `verify gate failed after N attempt(s) of M: none named`, so the run's Stuck reason hides that the verify hung, and no test reaches the case.
**Fix:** Add a `report.TimedOut` case to the `what` switch that names the timeout and the log path, and extend the existing stuck-reason test with a timed-out report.

### [LOW] Revive once-marker ignores the history-length episode boundary

**Location:** `internal/battenshed/innerrun.go:446` (`reviveDeadDriver`), `:415` (`callHalted`).
**Issue:** `callHalted` starts a new halt episode, and Warns again, when the child's history length differs from the halt-warned marker.
The revived marker is removed only when Call sees the child in a non-halted state, so a child that resumes and re-halts between two polls with a dead driver is a new episode to the Warn and the same episode to the revive.
The marker names this process's pid, so the revive is skipped and the reason falls back to `lyx loom start`.
**Fix:** Remove the revived marker wherever `callHalted` rewrites the halt-warned marker for a new episode, and add a row to `TestInnerRun_ARunningChildEndsTheReviveEpisode` or `TestInnerRun_HaltedRevivesADeadDriverOncePerEpisode`.

### [LOW] TestStrandNamed repeats covered rows and its keep reason is false

**Location:** `internal/reedengine/strand_test.go:530`.
**Issue:** Rows `LiveVisibleMatches`, `DormantWithoutPaneMatches` and `HiddenMatches` are the cases `TestAddStrandUnless_NamedStrandSkips` already asserts through the public `AddStrandUnless` (`Live`, `DormantPaneGone`, `Hidden`), and `TestAddStrandUnless_NoNamedStrandAdds` covers the absent and other-name rows.
The `//testtiming:keep` reason says its covering tests run this code without asserting it, which no longer holds.
**Fix:** Delete `TestStrandNamed`, adding a row to the public tests only for what they miss, namely that the first of several matching strands is returned.

### [LOW] Changed overview bullet keeps two sentences on one line

**Location:** `docs/overview.md:373` (the loom bullet in the module list).
**Issue:** The repository's `CLAUDE.md` requires one sentence per line in every `.md` file.
The diff changed this line (adding `resume` to the verb list) and left `... root alias `lyx start`). `internal/shedverbs` owns ...` on one line.
**Fix:** Break the line after the closing parenthesis and period of the first sentence.
