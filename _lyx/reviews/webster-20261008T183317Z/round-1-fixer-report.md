# Webster round 1 — fixer report

Every finding was validated against the code and the plan and fixed, one commit each.
The plan's verify list (`go build`, `go vet` under the integration, tmux and llm tags, `go test ./...` and `go test -tags integration ./...`) passes after the last fix, and `lyx loom lint-comments` is clean.

## Fixed

### F1 [BLOCKING] — count-uncapped retry loops around real spawns (`802aa0e17`)

The premise holds: `PATTERN-spawn-observability` requires an attempt-count bound on a retry loop around a real spawn, and `startupTickCap` already treats each reed capture and status as one.
The constraint wins over cards 4 and 6's window-only bound, as the review says.

- `send.go` gains `windowPollCap(window, minInterval)`: the polls paced at least `minInterval` apart that fit in the window, plus slack for the first poll and a cut last sleep, with a non-positive interval paced as `idlePollInitial`.
- `awaitIdleSession` also fails `ErrSessionBusy` at `windowPollCap(sendReadyTimeout, idlePollInitial)` polls.
  The review did not name this loop, but it has the same shape, so it gets the same bound.
- `awaitSettledBox` fails `ErrSubmissionNotLanded` at `windowPollCap(submitConfirmTimeout, SubmitSettleMS)` reads, so `submit_settle_ms: 0` is bounded too.
- `confirmSubmitted` stops at `windowPollCap(submitConfirmTimeout, idlePollInitial)` Enters, the 250 ms floor its doubling interval never goes below after the first read.
- Both not-landed messages now name the count reached, not a closed window.
- The gate's `unsentReprompt bool` becomes `unsentReprompts int`.
  The `maxUnsentReprompts`-th (5) consecutive busy or unlanded re-prompt at one boundary takes the existing "re-prompt send failed, ending the loop with the attempts spent so far" path.
  `wait.go`'s header, `handleGatedBoundary`'s doc comment and `doc.go`'s send paragraph say so.
- Tests (rows, no new test functions):
  - `TestSend_WaitsForIdleSession` gains a busy-pane row under `frozenClock`.
  - `TestRun_Send_ConfirmsSubmission` gains a never-settling row and an Enter-cap row under `frozenClock`, each pinning the exact count.
  - `TestGate_UnsentRepromptRetries` gains a row where a re-prompt that stays busy ends the loop at 15 s rather than at the 30 s deadline.
    I checked that this row fails with the cap raised to 1000 (it ended at 31 s).

### F2 [MEDIUM] — webster's reclaim lost the operator Warn (`3e57720e2`)

The premise holds: the deleted `removeStrandIfLive` logged the Warn at every reclaim site, including `RemoveRecoveryStrands`, and card 9 says to keep it.
`strand.go` gains `stopLeftoverStrand(stopper, guid)`, which logs the Warn and then calls `StopStrand`.
All six reclaim calls go through it: Master and recovery at entry, begin-batch, recover-spawn, recover-batch's terminal stop and `RemoveRecoveryStrands`.
Because `StopStrand` returns only an error, webster cannot tell beforehand whether the strand is live.
So the wording becomes "stopping a leftover strand before respawning it; if it is still live, an in-flight batch run restarts from state.json rather than resuming".
No test was added: the old Warn had no test, and this finding is not a coverage gap.

### F3 [LOW] — send config comments disagree with the code (`f8911b8c4`)

All three drifts are confirmed: `closeAt` is set before typing, the only `sendVerified` callers are prompt, skill-load and injected sends, and `claudeengine` paces the Enter of `/clear`, `/compact`, `/reload-plugins` and `/model` with `submitSettleMS`.
The comments in `template.yaml` and on the matching `Config` fields are reworded to match.

### F4 [LOW] — "never appeared" when nothing was typed (`bd21ddf46`)

The premise holds: an idle pane passes `awaitIdleSession` at once, and a `closeAt` cut to an already-passed run deadline breaks the replay loop before `playInputs`.
The `!typed` case now has its own message: the submit window, cut to the send's deadline, had closed before typing began, and nothing was typed.
It still wraps `ErrSubmissionNotLanded` and ends with the pane tail.
`TestRun_Send_ConfirmsSubmission` gains a `pastDeadline` row that sends through `sendWithin` with the deadline already passed.
The row asserts the new message, no keys typed and no "never appeared"; this path had no coverage before.

### F5 [NIT] — stale wait.go header (`cdf887bb2`)

The header now calls `abandonStartup` start's teardown for a provider that never became ready or never took its first input, matching its doc comment.

## Disputed

None.

## Deferred

None.
