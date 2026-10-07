---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/smoke_starttail_test.go:100
    summary: Comments the diff adds or changes still break mid-phrase or carry two sentences on a line in five files, so the round-1 sweep missed them (PATTERN-comment-line-breaks)
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/run_test.go:938
    summary: New untagged tests neither call t.Parallel nor name the process-global state that keeps them serial, and TestRun_Send_ConfirmsSubmission swaps the package-level inputSleep without naming it (PATTERN-test-speed)
  - id: F3
    severity: LOW
    class: design
    location: internal/webstercli/reset.go:109
    summary: The reset verb's fallback for a RemoveRecoveryStrands error that is not a RecoveryStrandRemoveError is unreachable, and its message names no way forward
  - id: F4
    severity: LOW
    class: consistency
    location: internal/orchengine/watcher_test.go:1184
    summary: Two test names now contradict what the tests assert, TestWatcher_AutoCompactionReloadsSkillsRoleAndPointer (no skills load) and TestSmokeStart_AttachTailAddsNoOperatorStrand (no attach tail)
  - id: F5
    severity: LOW
    class: scope
    location: internal/loomcli/start_watchdog_test.go:5
    summary: The file comment still frames the watchdog call by its gate position under --no-attach, a gate Card 14 removed
  - id: F6
    severity: NIT
    class: consistency
    location: internal/orchengine/watchloop.go:162
    summary: Run's loop declares done from w.Tick, shadowing the done channel that signals the stopping-record goroutine
---

Range: no `_lyx/loom/rework/round-*/` exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `95902574`), with `main` taken from `_lyx/shed/bugfix-sessions/status.json` `product.parent`.
The plan's verify commands all pass on HEAD: `go build ./...`, `go vet -tags llm ./...`, `go test ./...`, `go test -tags integration ./...`, and `go test -tags tmux` over loomcli, webstercli, orchcli, orchengine and shuttleengine.
All 18 cards are implemented as their Intent describes.
Card 1's settle is read from `shuttle.yaml` rather than a constant, which the decision record's "Added after Discussion (operator, 2026-10-07)" entry approves, so it is not a finding.

Focus verdicts:

- Focus 2 (`saveStateForStrand` and `WatcherStopping`): not a defect.
  `saveStateForStrand` copies the persisted `WatcherStopping` into every save of a found record, so a tick that loaded its state before the signal cannot clear it.
  The only writers are `recordStoppingOnCancel` and `recordExit("")`.
  `TestRun_RecordsStoppingBeforeTheInFlightTickFinishes` reads a turn end into a tick that is blocked while the context is cancelled, which forces that tick's `readingChanged` save.
  It then checks that the record holds while the lock is held, that it survives `Run`'s return, and that the next `Run` clears it.
- Focus 3 (orch stencil header comments): not a defect.
  `orch-template-reload.md` is rendered only for an auto-compaction, and the note-naming pointer after a watcher-run compaction uses another stencil.
  Its header now says the pointer follows `/reload-plugins` and that no skills are retyped.
  `orch-template-role.md` says skills are loaded only after `/clear` and that a compaction keeps them.
  Both match `startReload(…, true)` on both compaction entries and `reloadStep`'s skip of the skills step.
- Focus 1: F1 below.

### [BLOCKING] Residual fixed-column wraps in comments the diff adds or changes

**Location:** `internal/loomcli/smoke_starttail_test.go:100-103`, `internal/loomcli/smoke_test.go:587-590`, `internal/shedadapters/burler_test.go:894-898`, `internal/shuttleengine/attach_test.go:217-219`, `internal/shuttleengine/claudeengine/claudeengine.go:1-5`

**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line, with no mid-phrase breaks, in every new or changed Go comment.
These comments were written or changed by this diff, yet they still break inside a phrase or join two sentences on one line:

- `smoke_starttail_test.go` breaks at "found by the same / argv-signature scan" and "which is the / one property", and a line carries "…uses. This is what proves…".
- `smoke_test.go:588` carries "…success envelope. Only" and breaks before "awaitRunLockDeadline".
- `burler_test.go` breaks at "the round's own / review-parse entry" and "map identically to the / spawn path's".
- `attach_test.go` breaks at "only the live / strand" and "an attach or a / failed removal".
- `claudeengine.go`'s changed header line sits inside a sentence that runs on to "which is what makes the whole adapter / hermetically testable".

Round 1 fixed these files one at a time and missed these, so the finding is about the sweep, not any one file.

**Fix:** Rewrite each of these comment blocks as one sentence per line, breaking only at sentence ends or independent-clause boundaries.
Then grep the diff's added comment lines once more for a mid-line sentence end or a line ending mid-phrase.

### [BLOCKING] New untagged tests skip t.Parallel without naming global state

**Location:** `internal/shuttleengine/run_test.go` `TestRun_Send_ConfirmsSubmission`; `internal/shuttleengine/attach_test.go` `TestAttach_RemovesSupersededStrands`; `internal/shuttleengine/session_test.go` `TestRunner_ReloadPlugins_PlaysSequenceOnALiveShuttleStrand`; `internal/shuttleengine/config_test.go` `TestLoadConfig_SubmitSettleMS`; `internal/shuttleengine/claudeengine/session_test.go` `TestInputBoxText`, `TestReloadPluginsSequence`; `internal/shuttleengine/claudeengine/usage_test.go` `TestCompactedSince_CountsTurnEndsAfterTheBoundary`; `internal/loomcli/bootstrap_test.go` `TestStartEnvelopeFields_PinsSuccessEnvelope`; `internal/orchengine/watchloop_test.go` `TestRun_RecordsStoppingBeforeTheInFlightTickFinishes`

**Issue:** `PATTERN-test-speed` says a test calls `t.Parallel` unless it touches process-global state, and that a comment names that state.
Each of these new untagged tests does neither.
`TestRun_Send_ConfirmsSubmission` swaps the package-level `inputSleep`, a real process-global, with no comment naming it.
The others touch only per-test fixtures (temp dirs, their own fakes), so nothing keeps them serial.
Their files' older tests are serial too, but the background file calls that kind of legacy convention not one to match.
`TestStart_NeverAttaches` shows the right shape: it is serial and names `TMUX`.

**Fix:** Add `t.Parallel()` to each test above that touches no global state, and to its subtests where they have their own fixtures.
Give `TestRun_Send_ConfirmsSubmission` a comment saying it stays serial because it replaces the package-level `inputSleep`.

### [LOW] Unreachable reset fallback with no way forward

**Location:** `internal/webstercli/reset.go:107-113`

**Issue:** `RemoveRecoveryStrands` returns either nil or a `*RecoveryStrandRemoveError`, so the `!errors.As` branch can never run.
Even so, the branch builds a refusal message with no way forward.
That is code for a state the callee's own contract rules out, and it is a refusal shape `PATTERN-refusal-way-forward` would reject if it were ever reached.

**Fix:** Drop the fallback and refuse with the `errors.As` result directly, keeping the way-forward message the only one.

### [LOW] Test names that contradict their assertions

**Location:** `internal/orchengine/watcher_test.go:1184`, `internal/loomcli/smoke_starttail_test.go:43`

**Issue:** `TestWatcher_AutoCompactionReloadsSkillsRoleAndPointer` now asserts the plugins reload and then the pointer, with no skills load.
`TestSmokeStart_AttachTailAddsNoOperatorStrand` names an attach tail that Card 14 removed.
A test name reads as the sentence of what is expected, so both now say the wrong thing.

**Fix:** Rename them to `TestWatcher_AutoCompactionReloadsPluginsThenPointer` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`.

### [LOW] Stale gate framing in start_watchdog_test.go

**Location:** `internal/loomcli/start_watchdog_test.go:5-10`

**Issue:** The file comment says the call's gate position ("that it fires even under --no-attach") is asserted by the smoke tier.
With the attach gate gone, `--no-attach` is a no-op and no gate position remains.
The smoke test this diff rewrote now says the spawn fires on every start.

**Fix:** Reword the comment so the smoke tier pins that the watchdog spawn fires on every start, past the reed `Up` and strand branch the offline tier cannot reach.

### [NIT] Shadowed done in Watcher.Run

**Location:** `internal/orchengine/watchloop.go:162`

**Issue:** `Run` now declares a `done` channel for the stopping-record goroutine, and the loop's `done, err := w.Tick()` shadows it with a bool.
The code is correct, but a reader has to work out which `done` each line means.

**Fix:** Name the channel `runReturned`, and the goroutine's exit channel to match.
