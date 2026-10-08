---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/websterengine/strand.go:119
    summary: stopLeftoverStrand logs its leftover-strand Warn on every reclaim, including a strand that is not live, and shuttle's stop verb then logs a second Warn for a live one
  - id: F2
    severity: NIT
    class: consistency
    location: internal/shuttleengine/doc.go:166
    summary: the send paragraph restates the turnStartIdleOverride and confirmBackoffCap values as ten seconds and five seconds instead of naming the constants
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test -race` over the touched packages and `go test ./...` all pass.
The cards' mechanical groups were checked against the tree: the Delete group of card 10 leaves no reference to `StrandRemover`, `NewReedStrandRemover`, `BurlerRemover`, `fakeRemover` or webster's `removeStrandIfLive` (the remaining `removeStrandIfLive` is shuttle's own new helper in `stop.go`), `sendExtraEnters` is gone, and the `New`, `abandonStartup`, `reclaimEntryTimeStrands` and `RemoveRecoveryStrands` re-signings have no stale caller.
The only raw `RemoveStrand` callers left outside `shuttleengine` are reed's own verbs, orch, and `loomcli`'s loom-driver removal, which `PATTERN-shuttle-stop` names as its exception.

### [LOW] Leftover-strand Warn fires on every reclaim and doubles shuttle's own Warn

**Location:** `internal/websterengine/strand.go:119` (`stopLeftoverStrand`), called from `reclaimEntryTimeStrands`, `BeginBatch`, `recoverSpawn` and `RemoveRecoveryStrands`.
**Issue:** The deleted `removeStrandIfLive` logged its Warn only after reed reported the strand live, so the line meant a real agent was about to be killed.
`stopLeftoverStrand` logs before it calls `StopStrand`, which is a no-op for a strand that is not live.
`reclaimEntryTimeStrands` calls it for every recorded `MasterStrand`, so a normal re-entry over an already-dead master logs "stopping a leftover strand ... an in-flight batch run restarts from state.json", a warning the detached driver's log carries.
For a live strand the stop verb also logs `shuttle: stopping a live strand`, so the same kill gets two Warn lines.
The doc comment justifies the Warn by "a live leftover strand is a real agent killed mid-work", which holds only for the live case.
**Fix:** Log the webster-side Warn only when the strand is live, or let shuttle's single Warn stand and move the "restarts from state.json" context into it or into the caller's error path; either way a dead strand logs nothing.

### [NIT] Perishable constants restated in the send paragraph of doc.go

**Location:** `internal/shuttleengine/doc.go:166` and the lines after it.
**Issue:** The paragraph says an unmatched turn start releases "after the pane has read idle for ten seconds" and that the confirm interval "doubles up to five seconds".
Both numbers are the values of `turnStartIdleOverride` and `confirmBackoffCap`, so a retuned constant leaves the package doc wrong with nothing flagging it.
**Fix:** Name the constants (`turnStartIdleOverride`, `confirmBackoffCap`) instead of their values.

## Focus directive outcomes

None of these is a departure; each question is settled against the current code.

- `TestStartup_SendFailuresTearDown` (`internal/shuttleengine/startup_test.go:1146`) calls `t.Parallel` at the test and in its subtests.
  `TestRun_Send_ConfirmsSubmission` (`internal/shuttleengine/run_test.go:1066`) does the same at the test (line 1067) and in its subtests (line 1285).
  Neither needs a global-state comment.
  The new `TestSend_WaitsForIdleSession` is serial and carries its own comment naming `logcapture`'s process-global logger, as `PATTERN-test-speed` requires.
- `maxUnsentReprompts` no longer exists anywhere in the tree.
  The gate loop in `internal/shuttleengine/wait.go` ends a gate early on no count of busy or unlanded re-prompts: `handleGatedBoundary` sets `unsentReprompt` on `ErrSessionBusy` or `ErrSubmissionNotLanded`, spends no attempt, and the retry runs until the run deadline reaches `classifyDeadlineExpiry`.
  That matches Card 6's bound and `TestGate_UnsentRepromptRetries`, whose "stays undelivered" row asserts the loop runs to the 30 s deadline with no attempt spent.
- The `internal/shuttleengine/send.go` header names the context, the idle wait, the settle, submit confirmation and clear of an input-box send, and the gated loop's send and boundary-idle checks, which covers `sendWithin`, `boundaryIdle` and `startHoldReleased`; `ErrSubmissionNotLanded` sits under the confirmation and clear it names.
  It states what the file holds and is not padded, so it meets the repo's comment conventions.
