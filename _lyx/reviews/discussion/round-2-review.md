---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D4 (verifyrun cancel paragraph)
    summary: D4 does not say where the SIGTERM-first cancel lives, while today's group SIGKILL cancel is proc.ConfigureGroupKill, shared with gitexec and the lyxbin test kit
    fix: In D4, after the sentence on verifyrun.Run's cancel, add that the SIGTERM-first cancel with its own SIGKILL timer and group-empty poll is a new cancel used only by verifyrun (in internal/proc's platform files or verifyrun's own), and that proc.ConfigureGroupKill keeps its immediate group SIGKILL for gitexec and the lyxbin test kit; add the same to Scope Out as a bullet naming gitexec's cancel as unchanged.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (run start and DeliverPendingPrompt bullets)
    summary: D2's wait-then-resend waits on the runner's geometry slug while D1 refuses on the target strand's slug, so where they differ the wait returns at once and the resend spins with no backoff
    fix: In D2's run-start bullet and DeliverPendingPrompt bullet, replace waiting through the runner's waitWhileCovered with waiting on the slug D1's refusal read, carried on ErrPaused, so the wait and the refusal judge the same slug; add an acceptance criterion that a start whose target slug differs from the runner's waits on the target's slug.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (orch runner sentence)
    summary: D1 tells the orch runner the hub record path for D1's read through a new setter, yet every line that runner types is a run-start line that skips the read
    fix: In D1, delete the clause telling the orch runner the hub record path through a setter, and state that the orch runner stays told no geometry, so its run-start lines read uncovered; keep the sentence that lyx orch start keeps starting under a hub pause, and drop the orch runner from the every-production-runner claim.
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D8 (failed GUID lookup refusal)
    summary: D8's failed-lookup refusal tells Master to rerun once tender status shows no pause but not to end its turn, so Master may keep a turn running, polling status under the pause
    fix: In D8, make the failed-lookup refusal text open with end your turn now, as the main refusal does, before no resume line will come and the rerun instruction; extend the failed-lookup acceptance criterion to assert that text.
---

### [MEDIUM] D4 leaves the home of the SIGTERM-first cancel open

**Location:** `_lyx/discussion/decision-record.md` D4.
**Issue:** `verifyrun.run` (`internal/verifyrun/run.go:79`) gets its group kill from `proc.ConfigureGroupKill`, whose Linux body (`internal/proc/procgroup_linux.go:11`) sets `cmd.Cancel` to an immediate `SIGKILL` of `-pid`.
That function is also called by `internal/gitexec/gitexec.go:213` and `internal/testkit/lyxbin/lyxbin.go:157`.
D4 says only that "`verifyrun.Run`'s cancel" becomes SIGTERM-first "through a timer of `verifyrun`'s own".
`verifyrun` cannot signal a process group portably without platform files, so the natural edit is `ConfigureGroupKill` itself, which would give every cancelled git command a SIGTERM and a ten-second wait.
The Scope Out bullet names only the `verifyrun.Run` callers (batten deploy and `Command` rows, `cardverify`), which D4 lists correctly; it bounds nothing for the shared `proc` helper.
The rest of the focus question holds: `exec.Cmd.WaitDelay` does kill the shell alone, and only while `Wait` has not returned, so a group SIGKILL escalation does need its own timer.

### [LOW] D2's waits and D1's refusal judge different slugs

**Location:** `_lyx/discussion/decision-record.md` D2, run start and `DeliverPendingPrompt` bullets.
**Issue:** `waitWhileCovered` (`internal/shuttleengine/run.go:105`) waits on `r.pause.Slug`, the runner's slug.
D1 refuses on the target strand's slug and falls back to the runner's slug only when the name does not parse.
For a target slug that a slug entry covers and a runner told an empty slug, `WaitUncovered` returns at once and the resend is refused again, a loop with no backoff against `PATTERN-daemon-wakeups`.
Production runners mostly start strands in their own worktree, so the case is narrow, but the record keys D1 on the target precisely because runner and target can differ.

### [LOW] D1 arms the orch runner for a read none of its lines take

**Location:** `_lyx/discussion/decision-record.md` D1, the orch runner sentence.
**Issue:** The orch runner's only typed lines come from `runner.Start` (`internal/orchcli/start.go:81`); the rest of its use in `internal/orchcli/session.go` is reads and keys, and orch steps reach the orch through creel's runner.
D1 says the run-start lines skip the read, so the new setter and the hub path it carries serve no line, and the "every production runner is told one" claim adds an API with no caller effect.
The behaviour D1 wants, `lyx orch start` unheld, follows from the orch runner staying told no geometry.

### [LOW] D8's failed-lookup refusal does not tell Master to end its turn

**Location:** `_lyx/discussion/decision-record.md` D8.
**Issue:** The main refusal opens with "end your turn now"; the failed-lookup refusal says only "no resume line will come, so run the same verb again once `lyx tender status` shows no pause".
The Bound and Open risks rest on Master ending its turn and moving on only when next prompted, but the text Master reads never asks for that, so a Master may poll `lyx tender status` inside a running turn while the pause wants work stopped.

## Focus verdicts

- D1 and the idle wait: not a defect.
  `sendVerified` (`internal/shuttleengine/run.go:1457`) calls `awaitIdleSession` then `typeVerified`; D1 places the read after the idle wait and before typing, repeats it before each replay, keeps a pre-wait read only as an early exit, and the acceptance criterion "A pause added while a send waits for an idle session types nothing" pins it, matching the declared `PATTERN-pause-holds-prompts` text.
  `pauserecord.Read` takes no lock, so a read reached inside `SendConditioned` under tender revive's `Update` cannot deadlock.
- D6 and D7's orch identifier: not a defect.
  `State.Strand` is the orch strand's reed GUID (`internal/orchengine/state.go:79`); `recordOrchInEntry`, `settledWithoutInterrupt` and `queueResumeLine` in `internal/orchengine/pause.go` all match `Session.GUID` against it with `Orch` true, which is the shape D6 names.
- `lyx orch start` and a Verify entered under a covering pause: not a defect apart from F3.
  D1 and D2 leave the orch start unheld with an acceptance criterion, and D4 holds a Verify before its command with the `paused` marker, with an acceptance criterion.
- D4's escalation and callers: F1.
- D8's failed lookup: F4; the Bound and Open risks state that no line wakes Master.
