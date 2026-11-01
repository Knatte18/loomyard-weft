---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:115
    summary: 'D6 re-files the same adopt-fix slug each try, but a landed try leaves a done run record, so dobby gives the refiled task the done-slug start refusal and the loop never runs past try 2'
    fix: 'In D6''s slug bullet, make the slug unique per try (for example append the try number, -t<N>, and cut the subject so the whole slug still fits boardengine.MaxSlugLength, keeping the hash suffix rule for long subjects), and state that the fix task''s slug therefore never meets the done-slug start refusal (internal/battencli/dobbyreads.go sets StartRefusal from doneSlugRefusal for a released task whose run status reads done). Update D6''s acceptance criteria to pin that a second try after a landed first try starts. Alternatively state that the filing first clears the landed slug''s run directory and leftover task branch as doneSlugRefusal''s way forward describes, naming the function that does it; pick one and record it.'
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:120
    summary: 'A fix task halted on a machine failure keeps a run-form status (blocked or failed · row), so D8 reads it open; a still-red check after the operator''s re-run files none and waits for a landing that never comes, leaving the subject stuck'
    fix: 'In D6, change the pre-filing removal bullet (line 119) to remove the subject''s own fix task (open front task of its project whose on_land is adopt-check) when it stands draft or halted, through fronttask''s removal, before the open-front-task test at line 120; restrict line 120''s no-filing rule to an open front task of the worktree that is not the subject''s own halted fix task. Add an acceptance criterion: a hand lym adopt whose check is red while the subject''s fix task stands halted removes it and files a new fix task.'
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:156
    summary: 'fronttask''s removal starts with lym batten cancel --remove, which refuses nothing-left-to-cancel for a released fix task that never started and for a draft one a hand cancel --remove left, so the removal stops at step 1 and D5''s green removal and D6''s pre-filing removal never complete'
    fix: 'In D8''s removal bullet, state that step 1 runs lym batten cancel <slug> --remove only when the slug has a run claim, record or pair, and that cancel''s nothing-left refusal (internal/battencli/cancel.go refuseNothingLeft) counts as step 1 done, so the record drop and the entry removal follow. In D5''s failed-removal notice, keep lym board remove <slug> as the way forward and drop lym batten cancel <slug> --remove for that case. In D6, state that the filing does not go ahead while a removal it needed failed, and that its notice names lym board remove <slug>.'
  - id: F4
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:152
    summary: 'D8 has every dobby record any open front entry it sees and run its on_land action with no owner condition, so the old machine''s still-ticking dobby runs lym adopt check on a subject it no longer owns and can file fix tasks onto the shared board'
    fix: 'In D8''s record bullet, state that dobby records a front entry it sees with no record, and runs any recorded task''s on_land action, only while this machine is the effective owner of the fronted worktree''s subject (the beam''s owner, or the prime owner for the prime), read through boardengine''s owner read; a record on a machine that reads the subject owned elsewhere is dropped without running its action. State the same owner test at the start of lym adopt check <subject>, which then exits with a notice naming lym adopt <subject> on this machine. Add an acceptance criterion: a front task landing while a second machine''s dobby of the same worktree is live runs its action on the owner only.'
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:64
    summary: 'D4 stores the held slugs in the subject''s check record, but runCheck writes a fresh CheckRecord at its start and rewrites the whole record at each tier, so the list adopt wrote is wiped and the green end lifts nothing'
    fix: 'In D4, store the held slugs in a file of their own per subject beside the check record under fabricengine.AdoptScratchDir (for example holds.json, written through internal/state under its own lock), which adopt appends to and only the green end and a lift empty; state that the check reads it at its end and never rewrites it. Change D5''s green-end bullet and its subject-start rule (each subject whose record lists a hold still standing) to read that file. Alternatively state that runCheck carries the held slugs forward on every write; pick one and record it.'
  - id: F6
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:84
    summary: 'The declared PATTERN-adopt-hold says only the subject''s green check, or lym tender resume by hand, lifts a hold, but lym batten cancel lifts every pause entry of the slug''s own scope, HoldCaller included, and the record names no change'
    fix: 'Either add lym batten cancel to the declared PATTERN-adopt-hold text in both D4''s bound paragraph and the PATTERN changes section (for example "...; lym tender resume lifts it by hand, and lym batten cancel lifts the hold of the run it ends"), or add to D4 that cancel''s slug-pause lift (liftSlugPauses in internal/battencli/cancelcmd.go) skips HoldCaller entries and that the green end drops a held slug whose run is gone. Pick one and record it.'
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:147
    summary: 'D8 files the front task in one boardengine write of a released task, but the board''s draft rule makes every new task draft even when the payload sets a null status, so the stated write files a closed draft task'
    fix: 'In D8''s first paragraph, replace one boardengine write of a released task with the actual mechanism. Either fronttask''s write is exempt from the draft rule (applyDraftRule in internal/boardengine/draft.go), stated as a bound beside the run-agent guard bullet, or it is a create followed by a release write (null status). In the second case, state that a draft front task of fronttask''s own record is released by the next dobby tick or removed by D6''s pre-filing removal, so a filing that dies between the two writes is healed.'
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:171
    summary: 'The darn producer appends darnStopWayForward (lym loom decision add, then lym loom resume) to every stop''s halt reason, so the machine-failure stop names lym loom resume, against D6 and D7''s halted line'
    fix: 'In D9, state that for a stop on a condition the task body names, the darn producer (internal/loomshed/darnwrite.go) puts the stop report''s own way forward in the halt reason and appends no darnStopWayForward, and that the marker or report tells the two stops apart (for example a second marker file). Add to D9''s acceptance criterion that the halt reason of such a stop names the body''s way forward and not lym loom resume.'
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:138
    summary: 'A run cancelled on the old machine (records removed, entry draft) is not owned elsewhere, not landed, not held, paused or halted, so D7''s fall-through reports it running_here'
    fix: 'In D7, add a state, for example ended, before running_here, for a run this machine''s records held as not done before the reconcile whose entry after it reads draft or abandoned and whose run record is gone, naming the entry''s status; or extend landed''s reading to cover it under a different name. Add the state to D7''s acceptance criterion and the closed state list.'
  - id: F10
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:129
    summary: 'D7 never defines which entries make up runs; held released tasks and an unstarted fix task have no run record, so they are in neither movedRuns list and their state and row have no source'
    fix: 'In D7''s first paragraph, define runs as the union of movedRuns before the reconcile, movedRuns after it, the released tasks D4 holds and the subject''s open front task. State that an entry with no run record reports no row, and that a held released task reads waiting_for_check or held_failed_check and an unstarted fix task reads running_here (or a new state such as waiting_to_start).'
  - id: F11
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:217
    summary: 'D5''s criterion says no fix-task entry remains after a green end, but D5 removes the fix task only when halted or draft; a fix task released or running at the green stays and, under D8, holds every released task until it lands'
    fix: 'Either extend D5''s removal bullet (line 101) to remove the subject''s fix task in any open or draft state at a green end, through fronttask''s removal with F3''s nothing-left rule, cancelling a running one, or narrow the D5 acceptance criterion to say a halted or draft fix task is removed and a released or running one lands and runs its check again. Make the bullet and the criterion agree.'
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:162
    summary: 'D8''s bound covers only the command; a board edit to a released front task''s project, or lym board rename, moves which worktree it blocks and which subject''s check runs, and a rename makes the action run twice'
    fix: 'In D8''s payload-refusal bullet, add that boardengine refuses a change of project, kind or slug (rename) on an entry whose stored front is true, with a refusal naming fronttask''s removal and the re-run of lym adopt <subject>, and that a test reaches it through lym board upsert and lym board rename; extend D8''s bound sentence to say a board edit cannot move a front task to another worktree or slug.'
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:137
    summary: 'D7''s halted reads a batten run record state named halted, but shed''s states are running, paused, done, blocked, failed and awaiting'
    fix: 'In D7''s halted bullet, replace whose batten run record reads halted with whose batten run record''s state is blocked or failed (internal/shedengine/status.go).'
  - id: F14
    severity: LOW
    class: scope
    origin: handler
    location: _lym/discussion/decision-record.md:154
    summary: 'D8''s sentence on why dobby''s notify_on_land memory could not serve is a rejected alternative, which belongs in the support log'
    fix: 'Cut from line 154 the clause starting dobby''s notify_on_land memory could not serve, keeping the sentence on the record outliving a dobby restart; add to support-log.md''s Rejected alternatives a bullet saying dobby''s notify_on_land memory was rejected as the land-action record because it never remembers a run first seen past its Land row (internal/dobbyengine/landnotice.go).'
---

### [BLOCKING] Fix-task slug reuse meets the done-slug start refusal

**Location:** `_lym/discussion/decision-record.md:115`
**Issue:** Every try files `adopt-fix-<subject>`. Land prunes the board entry but keeps the run directory with state `done`, and `internal/battencli/dobbyreads.go:288-293` gives a released task over such a record `StartRefusal` from `doneSlugRefusal` (`internal/battencli/arm.go:865-869`: "has already completed ... or give the new task a different slug"). `startReady` only calls `heldStart` on it (`internal/dobbyengine/starts.go:93-96`). The second try never starts. Because it reads as an open front task, D8 also blocks every other start in the worktree. Every lens raised this.

### [BLOCKING] A halted fix task stays open and blocks a new filing

**Location:** `_lym/discussion/decision-record.md:120`
**Issue:** A darn stop is a Stuck verdict, so the entry reads `blocked · <row>`, which `IsRunStatus` accepts (`internal/boardengine/runlock.go:21-24`, `internal/shedengine/status.go:18-23`). D8 counts any run status as open (line 148). The operator re-runs `lym adopt <subject>` as D6 directs. If the check is still red, line 120 files nothing and tells the operator to wait until the halted task lands, which never happens. Every run of the subject stays held, and no task in the worktree starts. Only a green check clears this state. Every lens raised this.

### [BLOCKING] fronttask's removal fails at `lym batten cancel` when nothing is left

**Location:** `_lym/discussion/decision-record.md:156`
**Issue:** `refuseNothingLeft` (`internal/battencli/cancel.go:554-598`) refuses when no record, claim, pair, session, process or marker is left. A draft status is no claim (`internal/battencli/cancelcmd.go:496-497`). A released fix task that never started has nothing to cancel, and neither does a draft one left by a hand `lym batten cancel <slug> --remove`, which is the way forward D5's own notice names. D8 stops the removal at the failed step. D6's draft removal before filing (line 119) and D5's green-end removal then never finish, and the record does not say whether the filing goes ahead over the slug that is still taken.

### [BLOCKING] The land action runs on a machine that no longer owns the subject

**Location:** `_lym/discussion/decision-record.md:152`
**Issue:** The old machine releases nothing, so its dobby of the same worktree keeps ticking; `internal/dobbyengine/owned.go:14-17` only pauses runs owned elsewhere. Under D8's healing rule that dobby records the new owner's open fix task from the shared board. When the fix lands it reads the entry gone and spawns `lym adopt check <subject>` on the old machine. If that check is red, it files a fix task onto the shared board, which the owner's dobby then starts against a failure on the other machine, under a separate try count. Neither the record nor the action has an owner bound.

### [BLOCKING] The held-slug list is wiped by the check's own record writes

**Location:** `_lym/discussion/decision-record.md:64`
**Issue:** I verified this one myself. `runCheck` builds a fresh `CheckRecord{PID, StartTicks, Start, Gate, State}` and writes it through `writeCheck` (`state.WriteJSON`) at its start (`internal/adoptcli/check.go:143-145`). It writes the whole record again on each `save` and at `finish` (`check.go:161-173`). A held-slug list adopt wrote before the spawn is gone once the check starts, and so is a second adopt's list written mid-check. The green end then lifts nothing and the held runs stay held. D5's rule for which subjects start a check reads the same list.

### [BLOCKING] Cancel lifts a hold, against the declared PATTERN-adopt-hold

**Location:** `_lym/discussion/decision-record.md:84`
**Issue:** The declared entry reads "only its subject's green adopt check lifts it, never dobby; `lym tender resume` lifts it by hand". `lym batten cancel` runs `liftSlugPauses` (`internal/battencli/cancelcmd.go:320`), which lifts every live entry of the slug's own scope, a `HoldCaller` entry included. That third lifter is in the code the design keeps, and the record neither names it nor carves it out. The lenses rated this LOW. Instruction step 1 makes every departure from a listed constraint, judged by its declared text, BLOCKING, so it is raised here.

### [MEDIUM] "One write of a released task" against the draft rule

**Location:** `_lym/discussion/decision-record.md:147`
**Issue:** `applyDraftRule` (`internal/boardengine/draft.go:37-60`) gives every new task with an empty status `draft`, and `internal/boardengine/doc.go:245-248` says so "even when the payload sets `status` to null". A release is a second write. The filing D8 describes would produce a draft task, which D8 counts as closed and dobby does not start. D8's heal sees only open entries, so a death between two writes is not healed either.

### [MEDIUM] The machine-failure halt names `lym loom resume`

**Location:** `_lym/discussion/decision-record.md:171`
**Issue:** On a stop marker, `internal/loomshed/darnwrite.go:138-147` joins the stop report with `darnStopWayForward` (`darnwrite.go:30`: record the answer with `lym loom decision add`, then run `lym loom resume`). D6 requires the stop to name the re-run of `lym adopt <subject>` and never `lym loom resume`, and D7's `halted` line repeats the stop report's way forward. D9 leaves the appended text in place.

### [MEDIUM] A run cancelled on the old machine reads `running_here`

**Location:** `_lym/discussion/decision-record.md:138`
**Issue:** Cancel removes the run's records and writes `draft` (cancel help text, `clearRunClaim`). Such a run is in this machine's pre-reconcile list (`internal/adoptcli/adopt.go:186`) but not in the post-reconcile `movedRuns` (`adopt.go:216`). It is not owned elsewhere, not `landed` (its entry is present and not done), not held, not paused and not halted, so the order of D7's states lands on `running_here`. An `abandoned` entry ends up there too.

### [MEDIUM] D7's run list is undefined

**Location:** `_lym/discussion/decision-record.md:129`
**Issue:** "One item per run under the named subjects" names no source. `movedRuns` lists only entries holding a run record that is not done. The released tasks D4 holds and an unstarted fix task have none, so their `waiting_for_check` and `running_here` lines have no stated source and no row. D4's criterion that a released task gets a hold has no matching report.

### [MEDIUM] D5's acceptance criterion contradicts its removal rule

**Location:** `_lym/discussion/decision-record.md:217`
**Issue:** Line 101 removes only a halted or draft fix task. A hand re-run of adopt while the fix task is released or running starts a new check. If that check is green, the fix entry remains, and D8 holds the subject's released tasks until it lands. The criterion "after the green end no fix-task entry of the subject remains and dobby starts the subject's released tasks" is then false.

### [MEDIUM] Board edits move a front task's routing

**Location:** `_lym/discussion/decision-record.md:162`
**Issue:** A released front task holds no run lock, so `lym board upsert` may change its `project`. The task then blocks another worktree, and `adopt-check` runs for that worktree's subject. `lym board rename` keeps every field under the new slug (`internal/boardengine/doc.go`, Rename). The record, keyed by the old slug, reads gone and runs the action at once, and the renamed entry is healed as a new front task and runs it again when it lands. The rubric's attack-surface rule asks D8 to bound this routing edge, and it does not.

### [LOW] `halted` names no run-record state

**Location:** `_lym/discussion/decision-record.md:137`
**Issue:** The run-record states are `running`, `paused`, `done`, `blocked`, `failed` and `awaiting` (`internal/shedengine/status.go:18-23`). No record reads `halted`.

### [LOW] Rejected-alternative reasoning inside D8

**Location:** `_lym/discussion/decision-record.md:154`
**Issue:** Why `notify_on_land`'s memory could not serve is a rejected alternative, and per the record spec it belongs in the support log's Rejected alternatives. The requirement it carries, that the record outlives a dobby restart, stays in D8. The completeness-before-leanness test therefore holds.

## Focus verdicts

- **Fix-task lifecycle (D4, D5, D6, D8).** A re-run of `lym adopt` while the fix task stands released and unstarted leaves it unheld, as D4 says: it has no run record and carries `front`. Dobby starts it past `max_runs` only if the bypass comes before `startReady`'s `running >= MaxRuns` break (`starts.go:80-83`). D8 states that requirement, so it is not a defect. Three paths do leave a state where the fix task cannot start, or where an open front entry blocks the worktree: the second try (F1), a halted fix with a red re-check (F2), and a draft fix left by a hand `lym batten cancel --remove` (F3). F5 adds a fourth: the held runs are never lifted.
- **D7's closed state set.** `landed` can be produced: the pre-reconcile list plus the reconciled board, since Land's `PruneBoard` removes every done entry. `halted` can be produced from a `blocked` or `failed` record (F13 is about its wording). `running_here` misreads a run cancelled elsewhere (F9), and the population the states range over is undefined (F10).
- **Front-task writes.** The slug rule keeps one fix slug per subject within `MaxSlugLength` 32: 10 + 15 + 1 + 6. The only collision is a 22-character subject ending in a six-hex suffix that matches a long subject's hash, which is negligible. Upsert, upsert-batch and merge share one payload check site (`internal/boardengine/store.go:507,842,1021`), so D8's refusal mirroring `checkBeamPayload` binds all three. Rename is outside it (F12). The in-process write meets no run-agent guard, because only `boardcli` sets `Config.StrandName` (`internal/boardcli/cli.go:161`). D8 states this as a bound, and the parent authorised lym filing the task (creel #212), so the rubric's "files a task" bullet does not apply. `on_land` survives a dobby restart through fronttask's machine-local record, but the same healing record carries it to the wrong machine (F4). The one-write premise is false (F7).

## Rejected

- The interactions lens's note on D8's single write was routed outside its lens. It is kept once, as F7.
- The bound-breaker lens's possible slug collision between a 22-character subject and a long subject's hashed slug is not a defect: it needs a 24-bit hash match.
