---
verdict: APPROVED
---

The diff makes the three changes the Goal asks for, and I found no defect in it.

- **Fault 1.**
  - `Runner.SessionIdle` now takes the live strand's provider for a strand with no run record, as `Runner.Send` and `interruptEngine` do.
  - An empty tag resolves to the default engine, and an unknown tag returns the lookup error.
  - A guid that is neither a run nor a live strand gets the same refusal `Send` gives.
- **Fault 2.**
  - The Codex launch line appends `{"hook_event_name":"LymExit"}` between Codex and `exit`.
  - `ParseSessionSignals` reads it as a process-ending session end, so `settle` yields `dead` with cause `codex-exit`.
  - `sessionGone` sees it too, behind the output-files check the completion-signal rule requires.
  - The record carries no stamp or session id, so the parser's pairing and `stampEventTimes` are unaffected.
- **Fault 3.**
  - Both launch lines carry `-c 'sandbox_workspace_write.network_access=true'`, quoted through `sh.Quote`.
  - `Shell.AppendLine` is implemented for both dialects with correct quoting, and no other `Shell` implementer exists.
  - `doc.go` says what is opened and why.
  - The summary's account of the sandbox test is a claim I did not re-run.
- **Commits, PATTERN and docs.**
  - The commit messages match the diff.
  - No PATTERN change is declared or needed, and `PATTERN-shuttle-provider-seam` holds.
  - The `shuttleengine/doc.go` and `codexengine` doc updates agree with the code.

## Rejected

- Test lens, MEDIUM: no test takes a Codex exit record through the session-state fold to `dead`. The parser row in `TestParseSessionSignals` pins the signal, and the fold's `EndsProcess` rows in `sessionstate_test.go` pin the settle on that same generic signal. The fold is provider-neutral, so a third test over both halves would duplicate coverage.
- Test lens, LOW: no row for a recordless strand with an unregistered provider tag. The `engineFor` failure path is not new behavior, and the other recordless paths treat it the same way.
- Test lens, LOW: the recordless live-strand failure is asserted only as a non-nil error. `newFixture` writes no run record, so `TestRunner_SessionIdle_DeadStrandErrors` reaches the new branch. The refusal's wording is pinned for the shared helper at `run_test.go:1186`.
- Test lens, items 7 to 9, reported as "no finding" by the lens itself: the `Prepare` tests, the `skillload_test.go` assertion and the opened-session row all hold.
- Correctness lens, observation: `SessionIdle` reads reed's status twice for a recordless strand, once in `liveStrand` and once in `requireLiveStrand`. The second read is the safe one if the strand died between them, so I record no finding.
