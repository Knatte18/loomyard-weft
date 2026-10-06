---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/run.go:456
    summary: New and changed Go comments join independent clauses with a semicolon on one line, and two loomcli stub comments break mid-phrase, against PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/orchengine/watcher_test.go:1016
    summary: Two new or rewritten orch tests omit t.Parallel with no comment naming global state, against PATTERN-test-speed
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/orchengine/watcher.go:648
    summary: The rule that a persisted retry step with an empty reload_retry is read as the pointer step has no test, against PATTERN-test-economy
  - id: F4
    severity: BLOCKING
    class: scope
    location: internal/orchengine/watcher_test.go:908
    summary: The orch warn vocabulary (skill skipped with cause, skill load unverified) has no assertion, so TestWatcher_SkillSkipCauses passes if the watcher logs nothing, against PATTERN-test-economy
  - id: F5
    severity: NIT
    class: consistency
    location: internal/orchengine/state.go:222
    summary: ResetForFreshLaunch and Watcher.toIdle reset ReloadStep to the literal 0 while startReload uses ReloadStepSkills
---

Range: `_lyx/shed/skills-in-one-turn/status.json` gives `product.parent` `main`, `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` (four commits, one per card).

The implementation matches the plan on every question the focus directive raises.
`go build ./...`, `go vet` under the `llm` tag, and `go test` under no tag, `integration` and `tmux` pass for the plan's verify packages.
`grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal` is empty, and every card-4 Delete target (`Runner.LoadSkill`, both needles, `TestSkillLoadSequence`, `TestSkillUnknown`, the `shuttlefake` fields) is gone, not only the first.
Message text, the Skill-tool name and every transcript shape appear only in `claudeengine/skillload.go`.
The finding blocks below are the defects that remain.

### [BLOCKING] Comments break PATTERN-comment-line-breaks

**Location:** `internal/shuttleengine/run.go:456`, `internal/shuttleengine/run.go:488`, `internal/shuttleengine/claudeengine/skillload.go:79`, `internal/shuttleengine/doc.go:101`, `internal/orchengine/doc.go:183`, `internal/orchengine/doc.go:197`, `internal/orchengine/watcher.go:725`, `internal/orchengine/watcher_test.go:38`, `internal/loomcli/integration_driverbootstrap_test.go:50`, `internal/loomcli/integration_driverbootstrap_test.go:52`
**Issue:** Each listed line is new or changed by the diff and holds a semicolon followed by a clause with its own subject and verb (for example `...is confirmed unverified; none fails or hangs the launch.`), where the pattern requires a break at an independent-clause boundary.
The two `integration_driverbootstrap_test.go` lines also end mid-phrase (`answered by`, `takes the`), a fixed-column wrap carried into changed lines.
**Fix:** Break each line at the semicolon so the following clause starts its own line, and re-flow the two mid-phrase breaks so each sentence sits on one line.
Leave untouched lines as they are.

### [BLOCKING] New orch tests skip t.Parallel without naming global state

**Location:** `internal/orchengine/watcher_test.go:1016`, `internal/orchengine/watcher_test.go:1052`
**Issue:** `TestWatcher_ReloadRestartAroundTheRetryStep` (rewritten) and `TestWatcher_ReloadReadsAnOldStyleStepAsThePointer` (new) run on `newWatchEnv`, which holds only fakes and a temp dir, and neither calls `t.Parallel` nor carries a comment naming process-global state.
`PATTERN-test-speed` requires `t.Parallel` unless a comment names the global state touched.
**Fix:** Add `t.Parallel()` to both tests after confirming they pass under `go test -race -count=3`.

### [BLOCKING] Empty reload_retry on a persisted retry step has no test

**Location:** `internal/orchengine/watcher.go:648`
**Issue:** `reloadStep` reads a persisted retry step with an empty `ReloadRetry` as the pointer step, as card 3 states, and no test reaches that branch.
`TestWatcher_ReloadReadsAnOldStyleStepAsThePointer` covers only an old per-skill index.
`PATTERN-test-economy` names a behavior the diff adds with no coverage as a finding.
**Fix:** Extend `TestWatcher_ReloadReadsAnOldStyleStepAsThePointer` into a table with a row for `ReloadStep: ReloadStepRetry` and a nil `ReloadRetry`, resuming at the pointer.
This extends an existing test, so it adds no new function.

### [BLOCKING] Orch warn vocabulary is not asserted

**Location:** `internal/orchengine/watcher_test.go:908`
**Issue:** The shared decision `warn-vocabulary` makes the orch log one `skill skipped` Warn per skill with `skill` and `cause` (`unknown`, `not loaded`, `timeout`) and one `skill load unverified` Warn listing the skills.
`TestWatcher_SkillSkipCauses` is named for the causes yet checks only the calls and the step, so an implementation that logs nothing passes every row.
The shuttle side asserts the same vocabulary in `TestStartup_SkillLoadTurns`, so the orch side is the only uncovered half.
**Fix:** Add a `wantLogs` column to the `TestWatcher_SkillSkipCauses` rows, assert it against captured logger output as `TestStartup_SkillLoadTurns` does, and give the test the comment that names the process-global logger in place of `t.Parallel`.

### [NIT] ReloadStep reset to a literal 0

**Location:** `internal/orchengine/state.go:222`
**Issue:** `ResetForFreshLaunch` and `Watcher.toIdle` (`internal/orchengine/watcher.go:276`) assign the literal `0` to `ReloadStep`, while `startReload` assigns `ReloadStepSkills` and the `State.ReloadStep` comment names the constants.
**Fix:** Assign `ReloadStepSkills` at both sites.
