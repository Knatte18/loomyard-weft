```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: check that the claims the fix added or rewrote hold against the code at HEAD.

**[MEDIUM] D7 step 4: "any `Launch.PromptLine` is sent, and `RunState.PromptLine` and the prompt offset are persisted, as `launch` does for a new run"**
For the relaunch spec this persists no new prompt offset, so a later `Wait` starts reading before the limited turn end.
- Evidence: `launch` persists the prompt offset only inside `loadSkillsThenPrompt` (`internal/shuttleengine/run.go:848-860`). That function runs only when `launch.PromptLine != ""` (`run.go:777-781`).
- D7 step 2's spec has no skills, so `Prepare` leaves `PromptLine` empty (`claudeengine/claudeengine.go:281-284`). The continue line goes on the launch line, and following `launch` writes no offset.
- The record therefore keeps the original run's `PromptOffset`, which lies before the limited turn end. An attached `Wait` seeds its signal cursor from it (`internal/shuttleengine/wait.go:297-298`), so it can read the limited turn end again as one of the run's own.
- The text also doesn't say whether the original `RunState.PromptLine` is cleared. If it's kept, `DeliverPendingPrompt` (`run.go:900-925`) could type the task pointer into the relaunched session again.

**[MEDIUM] D7 step 4: "in one hold of the record's lock, which the stop verb's read and write of the record also take"**
This reads as a fact about the stop verb, but the stop verb takes no such lock today.
- `stopRecorded` reads and writes through `loadRunState` and `saveRunState`. Each takes `run.json.lock` separately (`rundir.go:202-215`, `stop.go:88-95`).
- `Run.Stop` holds only the in-process `recordMu` and writes without re-reading (`stop.go:43-56`).
- The text doesn't say the stop verb has to change so it holds the lock across its read and write. As written, the guarantee against a stop during the relaunch doesn't hold.

**[MEDIUM] D5: "make a switch take it, at most once per account before the limit hit's `blocked_until` rules it out"**
The "at most once per account" bound is wrong.
- `blocked_until` is written only when a session on that account hits the limit (D5).
- Until then, every limited worktree that runs D7's switch sees the forged low figures and can take the same exhausted account. That is one switch per such worktree, not one per account.
- The bound came from the reviewer's proposed fix.

Checked and holding:
- `ReplaceStrand` mints a new GUID (`internal/reedengine/strand.go:387`).
- The orphan sweep builds its set from every tracked strand (`run.go:1099-1102`), so a kept GUID keeps the run directory.
- "No forged input writes or clears a `blocked_until`" matches D5.

Tool calls:
- Bash grep of reedengine's `AddSpec` and `addStrandLocked`, and orchengine's `DecideStart`: my context had `ReplaceStrand` but not where the GUID is minted.
- Read `run.go:766-925`: my context lacked how `launch` sends `PromptLine` and persists the prompt offset.
