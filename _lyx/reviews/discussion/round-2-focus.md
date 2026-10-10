---
round: 2
focus:
  - "In decision-record.md D5, the until-stop loop writes `failed` to the status file after a stop when it still reads `running`: check that the Decision bounds this write against another holder of the run lock (a Go driver `lyx loom run`, an operator `lyx shed step`, a second loop), and against `internal/shedengine/status.go` giving `State` and `Error` to the engine alone. Settle whether an acceptance row covers a run another holder owns at the moment of the write."
  - "In D3, check what `--clear` can now let through: `internal/shedengine/status.go` documents `PauseRequested` as set only by an outside actor and written false only by the engine in the persist that records the paused state. Settle whether D3 states who may cancel an operator's pause, and an ownership class for the two new condition fields."
  - "In D2, settle how loop liveness is proved beyond a bare pid in the pid file (stale or reused pid after SIGKILL or reboot), and what makes the spawn exclusive when two `--until-stop` invocations both find no live loop."
  - "In D2, an envelope from a loop that ended unwatched is delivered by the next invocation: settle what makes that envelope current when the run may have moved on by `lyx loom resume`, a goto, or a Go driver or operator step in between."
  - "In D11, `pairteardown` must end the reed session before any worktree is removed (PATTERN-pair-teardown); settle whether a detached live loop can still step the run and spawn producer strands in the window before it is killed."
  - "Check that D1's closed `loop` key set names every key that D4, D7, D8 and the acceptance criteria place on the envelope (status-moved flag, stop-cause key, first-producer key, `envelope_path`, the transient re-step record), the stop cause of a bare pause, and the error-form keys of an `interrupted` stop."
  - "Check that the wording of the new PATTERN entry in D5 is true of exactly the code paths this task changes, given that Scope Out excludes the Go driver `lyx loom run`."
---

Each entry names a site and the question to settle; none caps a severity or pre-states a verdict.
