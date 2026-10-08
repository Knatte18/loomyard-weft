---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/websterengine/recoverbatch.go:188
    summary: stopLeftoverStrand runs and warns at sites that are not a leftover-strand respawn, including recoverSpawn with an empty prior guid
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/run.go:1138
    summary: the never-appeared message always reports 1+sendReplays attempts, though the submit window or the send deadline can end the loop after fewer
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/send.go:240
    summary: ErrSubmissionNotLanded doc says the text was typed, and the wait.go re-prompt comment says nothing is left in the box, though neither always holds
  - id: F4
    severity: NIT
    class: consistency
    location: internal/shuttleengine/template.yaml:7
    summary: send_ready_timeout_s and submit_confirm_timeout_s comments omit that they bind only engines with the idle reading or InputBoxReader
---

Range: `status.json` gives `product.parent` = `main` and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `6d9372c8f`).
`go build ./...`, `go vet -tags integration ./...`, and `go test` (untagged and `integration`) over `shuttleengine`, `websterengine`, `webstercli`, `shedadapters`, `shedrecipe`, `loomcli` and `testkit` pass.

The cards' own mechanical checks hold: no reference to `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive` or `removeStrandIfLive` (webster's) remains outside `internal/shuttleengine/stop.go`, whose own `removeStrandIfLive` is the one stop path.
`PATTERN-shuttle-stop` is in `PATTERN.md`; the remaining `RemoveStrand` callers are shuttle's own lifecycle teardowns, `reedcli` and `orchcli`.

### [MEDIUM] stopLeftoverStrand warns and stops at sites that are not a leftover respawn

**Location:** `internal/websterengine/recoverbatch.go:188` (and `:586`), `internal/websterengine/strand.go:118`
**Issue:** `stopLeftoverStrand` logs "stopping a leftover strand before respawning it; ... an in-flight batch run restarts from state.json" unconditionally, then calls `StopStrand`.
The deleted `removeStrandIfLive` warned only when reed reported the strand live, and `BeginBatch` still guards the call with `prior.StrandGUID != ""`.
Two sites now reach the helper without that precondition:
- `recoverSpawn` calls it for any non-nil `prior`.
  `RecoverSpawnOrAttach` reaches `recoverSpawn` with a terminal failed batch (`recoverbatch.go:312`), which is normally a fork batch with an empty `StrandGUID`.
  The result is a false Warn with `strandGUID=""` on the ordinary recover-batch path, and `StopStrand("")` runs `FindRun` over every run record, where a record whose `StrandGUID` is still empty would match and be marked stopped.
- `awaitTerminal` calls it after every terminal digest as routine cleanup, so each normal recovery completion logs a Warn that claims a batch run restarts from state.json.
`RemoveRecoveryStrands` skips an empty guid, so reset is unaffected.
The Warn's purpose (the operator is told a respawn kills an in-flight agent) holds only where a strand is stopped to be replaced.
**Fix:** guard `recoverSpawn`'s call with `prior.StrandGUID != ""` as `BeginBatch` does.
Keep the respawn Warn on the respawn sites (entry-time reclaim, begin-batch, recover-batch's prior strand, reset) and have `awaitTerminal` call `Stopper.StopStrand` directly, or word the Warn so it is true at every caller.

### [LOW] The never-appeared message misreports the attempt count

**Location:** `internal/shuttleengine/run.go:1138`
**Issue:** `sendVerified` ends the typing loop when `windowClosed` is true, in the outer `try` check or the inner attempt condition; the window is cut to the send's deadline.
The error still says "after %d attempt(s)" with `1+sendReplays`, and "the provider TUI likely swallowed the input", though only one typing, or part of one polling run, may have happened before a gate send's run deadline arrived.
The round-1 F4 fix covered only the case where nothing was typed.
**Fix:** count the typings actually played and say when the submit window or deadline ended the loop, so the message describes the path taken.

### [LOW] Two comments describe a narrower path than the code takes

**Location:** `internal/shuttleengine/send.go:240`, `internal/shuttleengine/wait.go:390`
**Issue:** The `ErrSubmissionNotLanded` doc says the text "was typed but never seen to be submitted", yet `sendVerified` also returns it when the window had closed before typing began and nothing was typed.
The `handleGatedBoundary` comment "Nothing of this send is left in the box" is also false when `clearUnlandedText` could not clear the box (a foreign draft, an engine without `InputBoxClearer`, a failed clear); the next tick then fails busy against that leftover until `maxUnsentReprompts` ends the loop.
**Fix:** reword the sentinel's doc to cover a send that typed nothing, and the wait.go comment to say the re-send may find the box still occupied and then fails busy.

### [NIT] Config comments omit which engines the new windows bind

**Location:** `internal/shuttleengine/template.yaml:7`, `internal/shuttleengine/config.go:34`
**Issue:** `send_ready_timeout_s` binds only an engine with the `SessionCycler` idle reading; without it the send keeps `requireReadyAgentPane` and no wait.
`submit_confirm_timeout_s` binds only an engine with `InputBoxReader`; without it the send has no window.
`doc.go` says both, the key comments do not, and the idle wait's "never past the run's deadline" is also missing.
**Fix:** add the capability condition to both key comments.

## Focus directive answers

1. Retry loops (`send.go`, `wait.go`): every loop around a real tmux call ends by attempt count as well as by time.
   `awaitIdleSession` stops at `windowPollCap`, `awaitSettledBox` at `maxReads`, `confirmSubmitted` at `maxEnters`, `sendVerified`'s typing loops at `sendReplays` and `sendVerifyAttempts`, and the unsent re-prompt at `maxUnsentReprompts`.
   Each is bounded above by its window, and `run_test.go` and `gate_test.go` drive them with `frozenClock`.
   Not a defect.
2. Webster reclaim: all five sites (entry-time master and recovery strands, begin-batch, recover-batch's prior strand, recover-batch's terminal stop, reset) log the Warn before the stop, including when `StopStrand` returns nil.
   The defect is the opposite one, in F1: the Warn also fires where no respawn follows.
3. Messages and comments: `sendVerified`'s nothing-typed message is correct; the other path messages and key comments are covered by F2, F3 and F4.

Round-1 files, read after this review was saved: F1 to F5 of round 1 are fixed as the fixer report says, and none regressed; the fixer deferred and disputed nothing.

Considered and not flagged: the new stop-failure errors in `BeginBatch`, `recoverSpawn` and the entry reclaim carry no way-forward text and no refusal-spec row.
The sibling failures they replace (a failed probe or removal) had neither, and a stop failure is an infrastructure error from reed or the run record rather than a guard, so it is not a `PATTERN-refusal-way-forward` refusal.
