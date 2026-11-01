---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:78
    summary: D4 holds every released not-started task under the subject on any adopt, so a re-run of lym adopt holds the released adopt-fix task itself and the fix loop deadlocks
    fix: In D4's second hold bullet, exclude a front task (an entry whose front field is true) from the hold, and add a D4 acceptance bullet that a re-run of lym adopt while adopt-fix-<subject> stands released with an empty status leaves it unheld and dobby starts it. D7's running_here line then stays true for the fix task.
    origin: lens:interactions
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:157
    summary: Held moved runs take max_runs slots in dobby's start step, so with max_runs or more held runs the released fix task never starts and nothing lifts the holds
    fix: Record the fact in D4 (dobbyengine slotsTaken counts every listed run not owned elsewhere, held runs included; startReady stops at max_runs; template default 3), then decide in D4 or D8 that a run covered by a HoldCaller entry takes no max_runs slot, or that dobby starts an open front task ahead of the slot limit; state the bound (one front task per worktree at a time, D6) and add an acceptance bullet with max_runs held runs and a filed fix task that dobby starts.
    origin: lens:existing-paths
  - id: F3
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:114
    summary: Batten's cancel leaves the cancelled fix task a draft entry with front set and its pair kept, so D8's start hold never lifts and D6 never files a fix again
    fix: Record the fact in D5 (lym batten cancel writes draft over the run status through clearRunClaim and keeps the pair unless --remove), and change D5's green-end bullet so the halted fix task is cancelled with --remove and its board entry removed through fronttask; state in D8 that a front task's start hold lifts when its entry is done, removed or no longer carries a run or released status. Add to the D5 acceptance bullet that after the green end no adopt-fix-<subject> entry remains and dobby starts the subject's released tasks.
    origin: lens:existing-paths
  - id: F4
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:143
    summary: D7's landed state reads a board entry done after the reconcile, but Land prunes every done entry, so a run that landed on the other machine has no entry and never reports landed
    fix: In D7's landed line, read a run as landed when this machine's records held it not done before the reconcile and its board entry is absent or done after it, citing the fact (battencli Land's PruneBoard calls boardengine Prune, which removes every done entry); update the matching Auto-mode assumption and the support log's Q11 to the same rule.
    origin: lens:walkthrough
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:128
    summary: Cutting the subject to fit the 32-character slug limit lets two subjects share one adopt-fix slug, so one subject's check skips filing, cancels or overwrites the other's fix task
    fix: In D6's slug bullet, state the limit as boardengine.MaxSlugLength and a rule that keeps the slug unique per subject, for example adopt-fix- plus the subject cut short enough to append a hyphen and a short hash of the full subject; and make D6's never-filed-twice test and D5's cancel find the fix task by its front field and project (or prime for root), never by slug alone.
    origin: lens:interactions
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:156
    summary: D6 and D8 file a task at runtime through boardengine but never say how that write meets the board's run-agent task guard, which judges only Config.StrandName set by boardcli from LYM_STRAND_NAME
    fix: Add to D8 that fronttask's boardengine write carries no strand name, so the run-agent task guard (boardengine checkTaskCreation) does not judge it, and state the bound - only fronttask files such a task, only a caller in its closed set (the adopt check now) calls it, and the parent authorised lym filing the fix task (D3, creel 212).
    origin: lens:existing-paths
  - id: F7
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:160
    summary: D8 refuses front and on_land only on lym board upsert, while upsert-batch and merge take the same entry payload, so the sole-writer rule of the declared PATTERN-front-task is unbound
    fix: Change D8's fourth bullet so boardengine refuses front and on_land on every payload write path (upsert, upsert-batch, merge), as checkBeamPayload does for beam, bar fronttask's own write; widen the D8 acceptance bullet to name all three verbs.
    origin: lens:existing-paths
  - id: F8
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:158
    summary: D8 copies notify_on_land's memory, which never remembers a run first seen past Land, so a fix task landing while its dobby is down or restarting never re-runs the check and the subject stays held with no notice
    fix: In D8, state this as a bound with its recovery, or close it - for example, the adopt check's record names the open fix task, and dobby or lym adopt start the check when that task's entry is gone and no check has run since; add an acceptance bullet for a fix task that lands while dobby is down.
    origin: lens:existing-paths
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:147
    summary: D7 reports a fix task halted on D9's machine-failure stop as running_here, misstating the one run the operator must act on
    fix: Add a halted state to D7's closed set (a run this machine owns whose batten run record is halted, naming the way forward from its stop), placed before running_here in the precedence order, and cover it in the D7 acceptance bullet.
    origin: lens:interactions
  - id: F10
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:128
    summary: D6 leaves the fix task's perch to the hub's darn default profile, but the parent required the fix with its perch, and the docs and speed profiles set code_max_bounces 0
    fix: In D6's slug bullet, file the fix task with an explicit profile whose darn code_max_bounces is at least 1 (light), dropping the light today aside.
    origin: handler
---

### [BLOCKING] A re-run of `lym adopt` holds the fix task and deadlocks the fix loop

**Location:** `_lym/discussion/decision-record.md:78`
**Issue:** D4's second hold bullet holds every task under an adopted subject that is released with an empty status, with no owner condition and no front-task exception.
D6 sends the operator to re-run `lym adopt <subject>`, and D5 starts a new check whenever the record lists a standing hold.
If `adopt-fix-<subject>` stands released and not yet started, the re-run holds it, the check runs over unchanged code and goes red, and D6 files nothing because an open fix task "is never filed twice".
Only `lym tender resume` breaks the loop, and D7 then reports the fix task as `waiting_for_check`, contradicting its own `running_here` line.
Dobby's start refuses a pause-covered slug (`internal/dobbyengine/starts.go` `readyToStart`).
Found by the interactions, bound-breaker and walkthrough lenses and by the handler.

### [BLOCKING] Held runs fill `max_runs`, so the fix task may never start

**Location:** `_lym/discussion/decision-record.md:157`
**Issue:** `slotsTaken` counts every listed run not recorded owned elsewhere, and a held run is owned here after the owner write, so it counts.
`startReady` breaks once `running >= cfg.MaxRuns`, and the template default is 3 (`internal/dobbyengine/template.yaml`).
A subject with three or more held moved runs and a red check gets its fix task filed and never started.
The runs wait for the check, and the check waits for the fix.
The record states no slot rule for held runs or front tasks.
Found by the existing-paths, bound-breaker and walkthrough lenses and by the handler.

### [BLOCKING] The green end's cancel leaves a draft front entry that wedges the worktree

**Location:** `_lym/discussion/decision-record.md:114`
**Issue:** `clearRunClaim` writes `StatusDraft`, and cancel keeps the pair unless `--remove` is set (`internal/battencli/cancelcmd.go`).
D8 lifts the start hold only once the front task "is done or removed".
After a green check cancels a halted fix task, the entry stays open with `front` set.
Dobby then starts no other task in the worktree, including the released tasks the green end has just lifted.
D6's "still open" test then refuses every later filing, and the slug cannot run again until `--remove`.
The record rests on an unchecked fact about what cancel leaves.
Found by the interactions, existing-paths, bound-breaker and walkthrough lenses and by the handler.

### [BLOCKING] `landed` can never fire, because Land prunes the done entry

**Location:** `_lym/discussion/decision-record.md:143`
**Issue:** D7 reads `landed` from a board entry that reads done after the reconcile.
Land's `PruneBoard` calls `boardengine` `Prune(true)`, which removes every done entry (`internal/battencli/land.go`, `internal/boardengine/board.go`).
So a run that landed on the other machine has no entry after the reconcile.
That is the board entry's own incident, the state the parent kept in D3, and the D7 acceptance bullet cannot be met as written.
D8 states the pruning itself.
Verified by the handler from the cited code.

### [MEDIUM] The slug cut can give two subjects one fix slug

**Location:** `_lym/discussion/decision-record.md:128`
**Issue:** `boardengine.MaxSlugLength` is 32, and `adopt-fix-` takes 10 characters.
Two beams sharing their first 22 characters therefore get one slug.
One subject's red check then sees the other's open fix task and files none.
One subject's green end can cancel the other's halted fix task, and a filing upserts onto the other's entry.
The record cites the limit only as "the board's slug limit" and states no uniqueness rule.

### [MEDIUM] The filing write's relation to the run-agent task guard is unstated

**Location:** `_lym/discussion/decision-record.md:156`
**Issue:** `checkTaskCreation` refuses a new task when `Config.StrandName` parses as a run agent's name.
Only `boardcli` sets that name, from `LYM_STRAND_NAME` (`internal/boardengine/taskguard.go`, `internal/boardcli/cli.go`).
D8 does not say which name `fronttask`'s in-process write carries, so the guard passes or refuses by accident of the spawner's environment.
The rubric's attack-surface bullet asks the Decision to state what the write gets past and what bounds it.
The parent authorised lym filing the task (D3), so the task-filing rubric bullet does not apply.

### [MEDIUM] The `front`/`on_land` refusal covers `upsert` only

**Location:** `_lym/discussion/decision-record.md:160`
**Issue:** `lym board upsert-batch` and `lym board merge` take the same entry payload.
The `beam` refusal D8 cites as its model is the engine-level `checkBeamPayload`, which covers every payload path.
Refusing on `upsert` alone leaves the declared `PATTERN-front-task` sole-writer rule unbound.

### [MEDIUM] `on_land` is lost when dobby is down across the fix task's Land

**Location:** `_lym/discussion/decision-record.md:158`
**Issue:** `notify_on_land`'s memory never remembers a run first seen past its Land row; note `landing-watch-limits` records this limit.
D3 requires that the check runs again by itself when the fix lands.
A dobby restart across Land leaves the subject held, with no check and no notice.
D8 states neither a bound nor a recovery.

### [MEDIUM] A halted fix task reports `running_here`

**Location:** `_lym/discussion/decision-record.md:147`
**Issue:** D7 puts the fix task under `running_here`.
A machine-failure stop halts the fix task, and that is the designed outcome (D6, D9) and the state the operator must act on.
D7's closed set has no state for it, which defeats D7's stated purpose of telling whether a run should run.

### [LOW] The fix task's perch rests on the hub's darn default

**Location:** `_lym/discussion/decision-record.md:128`
**Issue:** The parent ruled that the darn fixes "with its perch" (D3).
D6 files the fix task with no profile and leans on the hub default, which is `light` today per `lym board profiles`.
The `docs` and `speed` profiles set darn `code_max_bounces` 0, so a change to the default turns the perch off silently.

## Rejected

- Bound-breaker F7 (LOW, refusal names only `upsert`): the same defect as the existing-paths finding kept as F7 at MEDIUM. It is merged there, since the gap leaves a declared PATTERN rule unbound.

## Focus verdicts

- Focus 1, D4 against D6/D8 on a re-run: the hold covers the fix task and deadlocks; see F1.
- Focus 2, task filing: the rubric's task-filing bullet does not apply, because the parent ruled that lym files the fix task (D3, creel #212). The unstated guard relation is F6.
- Focus 3, `root`: not a defect. A task with no project has the prime as parent (`hubgeom.ParentOf`), adopting root brings the prime's dobby up through `orchcli.BeamUp`, and the prime's subject is `root`; the record names enough for Plan.
- Focus 4, slug: the cut does not keep subjects apart (F5). The limit is a board constant, `boardengine.MaxSlugLength` (32), which F5's fix names.
- Focus 5, attack surface:
  - `lym adopt check`: green lifts only the holds its own record lists.
  - The closed `on_land` table: bound stated.
  - The `HoldCaller` bypasses: stated in D4's bound.
  - D9: bound stated.
  - Defects: the cancel (F3), the upsert-only refusal (F7) and the task-guard bypass (F6).
- Focus 6, D3 against D7: the state set and precedence agree with D3, which carries `paused` as an Auto-mode assumption. The defects are `landed` (F4) and a halted fix task under `running_here` (F9).
- Focus 7, relocation: no finding.
  - D4's bound paragraph states behaviour Plan needs: push exemption, `--keep-local` and the hand lift.
  - Scope and the Auto-mode assumptions restate Decisions as their sections require.
  - Both PATTERN bullets pass the cap: `lym loom validate-discussion` exits ok.

## Process note

Before spawning the forks, I formed provisional judgments in my own reasoning while reading.
Step 1 asks the reader to judge nothing before the spawn, so this was a departure.
The fork prompts carried only the fixed boilerplate and lens text.
Each kept finding was reached independently by at least one lens.
