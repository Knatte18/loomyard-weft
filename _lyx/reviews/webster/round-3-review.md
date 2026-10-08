---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shuttleengine/stop_test.go:122
    summary: the guid form of the stop verb has no test for an unreadable run record or a failed record write, so stopRecorded's two refusal returns are unpinned
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/send.go:301
    summary: awaitSettledBox returns nil for a pair of agreeing reads taken after the submit window closed, so the first Enter can go out past closeAt and past the run deadline of a gate send
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`, since no `_lyx/loom/rework/` round exists and `_lyx/shed/shuttle-send/status.json` carries `product.parent: main`.
Judged against `_lyx/plan` cards 1–10.
Mechanical checks run: `go build ./...`, `go vet` under no tag, `integration`, `tmux` and `llm`, `go test ./...`, `go test -tags integration` over every touched package and `cmd/...`, and `go test -race` over `shuttleengine` and `burlerengine`; all passed.
Per-card mechanical checks: card 4's `Delete sendExtraEnters`, card 8's `burlerengine.New` re-signing (every caller compiles) and card 10's deletions (a repository grep for `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive` and `sendExtraEnters` finds nothing, and `removeStrandIfLive` survives only in `shuttleengine/stop.go`, where card 7 defines it) all hold.

### [MEDIUM] The guid form of the stop verb has no test for its two refusal paths

**Location:** `internal/shuttleengine/stop_test.go:122`, covering `internal/shuttleengine/stop.go:67-81`.
**Issue:** Card 7 says `stop_test.go` "tables both forms against `fakeReed`" for, among others, "record-write failure returning the error with no removal".
`TestStop_FailedRecordWriteRemovesNothingAndRestoresTheHandle` covers that only for `Run.Stop`.
`Runner.StopStrand` → `stopRecorded` has two `Errorf` returns (an unreadable `run.json`, a failed `saveRunState`) that the completion-signal tripwire audits as `stopRecorded [Errorf] x2`, and no test reaches either.
Webster's reclaims, attach (`removeSupersededStrands`) and the docs ("a stop that cannot be recorded fails the reclaim") all rest on exactly these returns, and webster's own tests fake the stopper, so nothing pins the real behavior: an error returned with the strand left live and `run.json` untouched.
The omission is a behavior the diff adds with no coverage (`PATTERN-test-economy`).
**Fix:** Add two rows to the guid form: a run directory whose `run.json` is unreadable, and one whose record cannot be rewritten; assert the error and `len(fake.RemoveStrandCalls) == 0`.
`TestStop_FailedRecordWriteRemovesNothingAndRestoresTheHandle` can stay the handle-form twin, so these are rows or one small guid-form test, not a second table.

### [LOW] The first Enter can go out after the submit window has closed

**Location:** `internal/shuttleengine/send.go:301` (`awaitSettledBox`) into `send.go:324` (`confirmSubmitted`).
**Issue:** `awaitSettledBox` checks `windowClosed` before each sleep, but returns nil as soon as the read taken after that sleep agrees with the previous one, without checking the window again.
`confirmSubmitted` then sends its first Enter unconditionally.
When the sleep and read carry the clock past `closeAt`, the Enter lands up to one `submit_settle_ms` plus one capture after the window closed; for a gate send the window is cut to `run.deadline`, so the Enter can follow the run deadline.
Card 4 says the window caps how long and therefore how many Enters go, and its Bound lists the overshoots (the last confirm read, the clear with its readings, `paneTail`); an Enter past the close is not among them.
The consequence is small — the Enter goes into a box that holds the settled sent text, and a confirm read follows — but it is a send after the window the plan says bounds it.
**Fix:** Either make `awaitSettledBox` return `ErrSubmissionNotLanded` when the window has closed by the time two reads agree, or add this Enter to the Bound the card and `doc.go` state.
