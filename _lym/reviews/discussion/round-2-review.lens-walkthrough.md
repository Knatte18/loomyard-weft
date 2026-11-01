```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case, feeding each step what the previous step produces in the code.

**Per decision: the case and whether its purpose is met**
- **D1:** `lym adopt payments` on Neve while OSL-1046 owns `payments`. The steps run in the stated order: reconcile, local-commit stop, pair add, holds, owner write, `BeamUp`, check, report. **Met.**
- **D2:** the prime's code branch is `diverged` after the reconcile. It becomes a warning naming `lym fabric pull`, and adopt goes on. **Met.**
- **D3:** the parent's rules. They are carried out through D4 to D9, so it is judged there. **Met as a record.**
- **D4:** a moved run `dobby-land-notice-needs-flag` that OSL-1046 owns, plus a released task under `payments`. The hold is written before the owner write. Dobby's `startReady` reads `PauseCovers` and starts nothing (`internal/dobbyengine/starts.go:87-98`). **Met.**
- **D5, green end with a halted fix task:** `cancel --remove` clears the `blocked · Darn-Write` claim, then the record is dropped and the entry removed. **Met.**
- **D5, green end with a draft fix task left by a hand `lym batten cancel <fix> --remove`:** `cancel` refuses because nothing is left, so the removal stops at step 1. **Not met** (F4).
- **D6, try 2 after try 1 landed:** the refiled slug carries a done-slug start refusal. **Not met** (F1).
- **D6, machine-failure halt, then a hand re-run whose check is still red:** the halted task stands open and blocks a new filing. **Not met** (F2).
- **D6, filing with "one write of a released task":** the draft rule turns the new task draft. **Not met** (F3).
- **D7, a run cancelled on the old machine:** it reads `running_here`. **Not met** (F5).
- **D7, a held released task with no run record:** it is in no list. **Not met** (F6).
- **D7, a fix task halted on a machine failure:** it can read `halted`, but the run record has no `halted` state (F7). Its way forward contradicts D6 (F8).
- **D8, the fix task lands on Neve while OSL-1046's dobby of the same worktree is alive:** the action also runs on OSL-1046. **Not met** (F9).
- **D8, `max_runs` held runs listed and the front task filed:** it starts only if the front task skips the `running >= MaxRuns` break. D8 states that it does. **Met.**
- **D9, darn writer with a stop condition in the task body:** the marker is written and the verdict is Stuck. **Met**, apart from the way forward (F8).

**Findings**

**F1 — BLOCKING, design, D6 slug rule and fix loop.** The retry slug cannot start again once an earlier try with that slug landed, so the loop never gets past try 2.
- Walk:
  1. Try 1 files `adopt-fix-payments` and it lands.
  2. Land's `PruneBoard` removes the entry, but the run's status file stays at `done`.
  3. The `adopt-check` action runs the check, it is red, and the same slug is filed again.
  4. Dobby's view of a released task sets `StartRefusal` from the done run status. `startReady` then only calls `heldStart`, and the task never starts.
- `adopt_fix_tries` above 1 is unreachable.
- Evidence: `internal/battencli/dobbyreads.go:288-293`, `internal/battencli/arm.go:865-869` ("has already completed … give the new task a different slug"), `internal/dobbyengine/starts.go:91-94`.

**F2 — BLOCKING, design, D6 and D8 (a halted fix task stands open).** A machine-failure halt leaves the subject with no way to run a new fix.
- Walk:
  1. The darn stop is a Stuck verdict, so the entry's status is `blocked · <row>`, which `IsRunStatus` accepts. D8 counts that as open.
  2. The operator re-runs `lym adopt payments`, and the check is still red, for example a code failure under the settled machine.
  3. D6 sees an open front task, files none, and its notice says to re-run once the task lands.
  4. The halted task never lands, since its way forward is the re-run.
  5. The holds stand and dobby starts no other task. Only a green check removes the halted task.
- Evidence: `internal/boardengine/runlock.go:17-29`, `internal/shedengine/status.go:21-22`, decision record D6 ("While any front task stands open … files none"), D8 open definition.

**F3 — MEDIUM, design, D8 (the filing is "one boardengine write of a released task").** The board's draft rule makes the stated single write produce a draft task.
- `applyDraftRule` sets `draft` on every entry of kind task with an empty status that was not a task before the write, even when the payload's status is null.
- `released()` requires an empty status, and a draft front task is closed under D8.
- Followed as written, the fix task never starts. The filing needs a second, releasing write, which opens a crash window D8's healing does not cover since it records only open entries, or it needs a draft-rule exemption. The record states neither.
- Evidence: `internal/boardengine/draft.go:37-60`, `internal/boardengine/doc.go:245-249`, `internal/dobbyengine/starts.go:42-44`.

**F4 — MEDIUM, design, D8 removal step 1 (D5 green end and D6's draft removal).** `fronttask`'s removal cannot get past `lym batten cancel` when the fix task has nothing left to cancel.
- `lym batten cancel <slug> --remove` refuses with "nothing of <slug> is left to cancel" when no record, claim, pair, session, process or marker is left.
- That is the case for a draft fix task after a hand `cancel --remove`, and for a released fix task that never started.
- Removal step 1 then fails every time, so the entry stays. D6's "draft fix task … removed before the filing" cannot finish, and the record does not say whether filing goes on over a slug that is already taken.
- D5's notice names `lym batten cancel <slug> --remove`, which refuses again.
- Evidence: `internal/battencli/cancel.go:554-598`, `internal/battencli/cancelcmd.go:489-503`.

**F5 — MEDIUM, design, D7 states.** A run cancelled on the old machine reads `running_here`.
- Walk:
  1. The run is not done in this machine's records before the reconcile.
  2. On OSL-1046 a cancel removed its records and left the entry `draft`.
  3. After the reconcile it is not `owned_elsewhere`, and not `landed` because the entry is present and not done.
  4. It is not held, because the post-reconcile `movedRuns` drops it. It is not `paused`, and not `halted` because it has no record.
  5. So the precedence lands on `running_here`, which is false.
- The same happens for an `abandoned` entry.
- Evidence: `internal/battencli/cancelcmd.go:52-56` (cancel removes the records and writes draft), `internal/adoptcli/adopt.go:186,216` (two `movedRuns` reads), decision record D7.

**F6 — MEDIUM, design, D7 `runs` population.** D7 never says which runs make up the list, so held released tasks have no source.
- D4 holds released tasks that have no run record, but `movedRuns` lists only entries holding a batten run record that is not done.
- A held released task, and an unstarted fix task D7 calls `running_here`, are therefore in neither list. Their "row" has no run record to read from.
- Evidence: `internal/adoptcli/doc.go` (`movedRuns` paragraph), decision record D7 ("one item per run under the named subjects").

**F7 — LOW, consistency, D7 `halted`.** D7 says "batten run record reads halted", but shed has no `halted` state; a halt is `blocked` or `failed`. The record should name the record states it maps to `halted`.
- Evidence: `internal/shedengine/status.go:18-23`.

**F8 — MEDIUM, design, D9 and D6 way forward.** The halt reason will name `lym loom resume`, which D6 forbids.
- D6 requires the machine-failure stop's way forward to be a re-run of `lym adopt <subject>`, "never `lym loom resume`".
- The darn producer always appends `darnStopWayForward` ("lym loom decision add … lym loom resume") to the stop report in the halt reason.
- D9 does not change that, so the halt and D7's `halted` line name `lym loom resume`.
- Evidence: `internal/loomshed/darnwrite.go:30,138-147`.

**F9 — BLOCKING, design, D8 land action across machines.** The old machine's dobby also runs the `adopt-check` action when the fix task lands.
- Walk:
  1. Neve files and runs the fix task.
  2. OSL-1046's dobby of the same worktree is still alive, since the old machine "releases nothing".
  3. Under D8's healing rule it records the open front entry it sees with no record.
  4. When Land prunes the entry, it reads gone and spawns `lym adopt check payments` on OSL-1046.
  5. A red check there files another fix task on the shared board for a subject Neve owns.
- D8 states no owner bound on the recording or on the action.
- Evidence: decision record D8 (recording and action bullets), `internal/dobbyengine/owned.go:14-17` (a dobby keeps ticking over worktrees owned elsewhere).

**Tool calls:** none.
