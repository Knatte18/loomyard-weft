---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/webstercli/cli_tmux_test.go:1
    summary: Go comments the range adds or moves are still hard-wrapped at a column, breaking sentences mid-phrase, against PATTERN-comment-line-breaks
  - id: F2
    severity: LOW
    class: consistency
    location: internal/reedcli/watchdog.go:1
    summary: Comments re-broken in round 1 still split one sentence across lines at non-clause boundaries (after an em dash, a colon, before a parenthetical, inside a list)
  - id: F3
    severity: LOW
    class: consistency
    location: internal/reedengine/overlay.go:217
    summary: EndSessionByName doc says the server's socket file is removed, but the removal is a bounded best-effort wait that leaves the file in place silently
  - id: F4
    severity: LOW
    class: scope
    location: internal/reedengine/socketfile_test.go:1
    summary: removeSocketFileOnceGone's wait-elapsed branch has no test; only the success path is covered, by the pairteardown integration test
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/reedcli/smoke_selvage_keepalive_test.go:174
    summary: TestSmokeSelvageSurvivesCtrlCAtIdlePrompt sends Ctrl-C before Selvage's shell reaches its prompt and fails about one run in five, failing the verify gate
---

Focus item 1 (socket-file removal on the engine-less path) is sound in its core:
`endSessionByNameVia` runs `kill-server`, then `removeSocketFileOnceGone` retries `removeStaleSocket` every `processExitPoll` for up to `socketGoneWait`, a removal error is logged at Debug and never fails the teardown, and `teardown_integration_test.go` asserts the file is gone (it passes under `-tags integration,tmux`).
Unlike `ensureServerGoneLocked` it has no server pid to wait on, so polling the socket is the right substitute.
The gaps are in the doc (F3) and in coverage of the elapsed-wait branch (F4).

Focus item 2 (comment line breaks): the eight named files are mostly clean after round 1, but round 1's fix covered only part of the range (F1, F2).

### [BLOCKING] Hard-wrapped Go comments remain in the range

**Location:** `internal/webstercli/cli_tmux_test.go` (the moved `TestRunCLIIn_StandalonePreRun_ReachesRunsOwnValidationGate` and `tearDownStandaloneReed` comments), `internal/webstercli/cli_integration_test.go` (file header), `internal/reedcli/watchdog_integration_test.go`, `internal/reedcli/watchdogreap_integration_test.go`, `internal/reedcli/smoke_resume_test.go`, `internal/reedcli/smoke_test.go`, `internal/reedcli/smoke_dotfill_test.go`, `internal/reedengine/contract_integration_test.go`, `internal/reedengine/ensuresession_integration_test.go`, `internal/reedengine/mouse_boot_integration_test.go`, `internal/shuttlecli/modelpin_test.go`, `internal/shuttleengine/claudeengine/testmain_test.go`, `internal/lyxcwd/enforcement_test.go`, `internal/burlerengine/smoke_cluster_test.go`.
**Issue:** each file has comment lines the range adds that wrap a sentence at a fixed column (a line ends mid-phrase and the next starts lowercase), which `PATTERN-comment-line-breaks` and `CLAUDE.md` forbid.
Comments that moved from `cli_integration_test.go` into `cli_tmux_test.go` appear as added lines and keep their old column wrapping.
**Fix:** re-break each into one sentence per line, joining the wrapped lines and changing no wording.

### [LOW] Round-1 re-breaks still split sentences at non-clause boundaries

**Location:** `internal/reedcli/watchdog.go` (the `loop:` comment, break after the em dash), `cmd/testtiming/main.go` (the `-redundancy` paragraph split inside one sentence's list, and the break before `parses the JSON event stream`), `internal/gitkit/reexecguard.go` (break before the parenthetical), `internal/reedengine/doc.go` (break before `and removed only when`).
**Issue:** the breaks fall inside one sentence at points that are not an independent-clause boundary or a semicolon, so they are column-style wraps in disguise.
**Fix:** join each sentence onto one line; the comments have no column limit.

### [LOW] EndSessionByName's doc overstates the socket-file removal

**Location:** `internal/reedengine/overlay.go:217`
**Issue:** the doc says the function kills the server "removing that server's socket file".
The removal is best-effort: it polls for `socketGoneWait`, and a file still answered after that stays, returns nil and logs nothing.
**Fix:** state the bounded best-effort removal in the doc comment.

### [MEDIUM] The Selvage idle-prompt Ctrl-C test races the shell's startup

**Location:** `internal/reedcli/smoke_selvage_keepalive_test.go` (`TestSmokeSelvageSurvivesCtrlCAtIdlePrompt`)
**Issue:** the verify gate failed on this `tmux`-tier test (`Selvage pane %1 not alive after Ctrl-C at its idle prompt`), and it fails about one run in five when run alone.
Bash ignores SIGINT only at its prompt, and the test sends Ctrl-C right after `up` returns, so a Ctrl-C landing during shell startup kills the pane.
**Fix:** wait for the Selvage pane to render its prompt before sending Ctrl-C.

### [LOW] The elapsed-wait branch of removeSocketFileOnceGone is untested

**Location:** `internal/reedengine/socketfile_test.go`
**Issue:** `removeSocketFileOnceGone` is covered only on its success path, through `TestRun_TaskWorktreeRemovedByHandEndsTheSessionByName`.
The branch where the wait elapses with a live listener, leaving the file and returning nil, has no test, and `wait` is injectable so it costs nothing to cover.
**Fix:** add a table-driven test over a stale socket (removed) and a live listener with a short wait (kept, no error).
