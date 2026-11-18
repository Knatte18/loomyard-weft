```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run end to end on one concrete case.

**Findings**

- **BLOCKING, D7: the restart cannot run through the stop verb inside the same run.**
  Case: a writer in task worktree W on account A hits its limit, and B is eligible.
  Shuttle's `Run.Stop` records the run as stopped (`markedStopped`, `stopOutcome` in `internal/shuttleengine/stop.go`). The waiter's `Run` then ends `runOutcomeStopped` through the completion-signal path.
  The record then says shuttle "starts it again as a resume of the same session id". It names no step that hands the producer's `Wait` the new strand or run.
  So the producer waiting on the run would see a stopped run whose outputs are missing, not a continuation on B.
  The input of step 2 (restart) is not what step 1 (stop verb) produces.
  The record must say either that the restart happens inside the same `Run` without the stop verb's record write, or how the caller learns of the new run.
  The constraint naming the stop verb for the restart makes this sharper.
- **MEDIUM, D7/D5: a session launched with no account has none to switch from or to mark blocked.**
  Case: the prime with no pin and no entry, or a worktree made before the change, on `~/.claude`, which is also listed as account X.
  The orch hits its limit. D5 records `blocked_until` on "its account", but the session resolved no account, so nothing is recorded.
  D7 picks among "the other candidates", other than nothing, so it can pick X. X now differs from the session's launched "none", so it restarts and resumes in the same directory, still limited.
  The record needs a rule that maps a launch with no account to the listed account whose directory the session ran under: the `env` value or the default, compared cleaned.
- **MEDIUM, D7: the record never says where a session's launched account is kept.**
  D7's concurrency rule compares the entry with "the one the session was launched under", and D5's `blocked_until` needs the account of the limited session.
  The launch line holds only the exported directory, and the record names no field in the run record or spec that keeps the launched account name for the switch call to read.
  Without it, the switch step has no input to compare against.
- **LOW, D3: picking for a pinned worktree skews the counter.**
  Case: hub pin P, weights 2 and 1. Every task Worktree-Create picks and advances A or B, though every session runs on P.
  When the pin is removed, the counts reflect picks never used. This is harmless to the share going forward, but the stated purpose ("entry stands ready") costs counter accuracy.
- **LOW, D8: the session-end usage record can count the turns before the switch twice.**
  Case: D7's switch stops the strand on A, and the resumed run on B ends later.
  If the stopped run writes a session-end usage record, both records read a transcript holding the A turns: the A file, then the B copy.
  `LocateSessions`' dedup covers `tools/tokencount`, not two separate usage records.

**Decisions walked, held**

- D1: machine file listing A and B with `account_pins.beams: A`; the tracked hub `claude.yaml` holding `accounts` is refused. Met.
- D2: the file is wiped; an entry is missing, so the launch exports nothing; a resume finds its transcript under the default through D6. Met.
- D3: weights 2 and 1 give A, B, A, A, B, A by lowest count over weight with ties to the earlier entry. Met.
- D4: W's entry is B on `/home/u/.claude-b`. The engine's command carries the export inside `Cmd`, after reed's prepended `env` exports, and reed's relaunch replays it. Met.
- D5: B's session settings carry the status line with name B; the verb merges `five_hour` 80% into `usage/B.json`, and the next pick passes over B at threshold 75. Met.
- D6: a resume of S in W, transcript under A's `projects/<enc W>/`, resolved to B. The copy is made, then `CheckResume` stats it under B. Met, given a matching cwd encoding.
- D7: blocked by the findings above.
- D8: `LocateSessions(W)` finds S under A and B and returns the B copy, which is newer and holds every turn. Met.
- D9: the orch resume goes through `spec.ResumeSessionID` (`internal/orchcli/start.go`), so Prepare's copy covers it. Met.
- D10: the manual steps follow D1, D4 and D6 in order. Met.

**Tool calls**

- I grepped orchcli, tenderengine and orchengine for the resume path, because the context lacked whether the orch resume goes through the engine's `Prepare`.
- I grepped `internal/shuttleengine/apierrorretry.go` and `stop.go` for the function shapes, because the context lacked whether the stop verb ends the run the waiter holds.
