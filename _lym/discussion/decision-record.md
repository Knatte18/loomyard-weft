# Decision record — machine-switch

## Goal

Taking a beam, or root, over onto this machine is one verb, `lym adopt`, that leaves every run under it in a known state.
It syncs, adopts, settles conflicts and starts each beam's dobby and orch.
The runs it takes over run here once the adopt check of their subject is green: they wait for the check and start by themselves on green.
A failed check files a darn fix task in that subject, the only run that starts; when the fix lands the check runs again, up to a configured number of tries, then the failure goes to the operator.
It prints one line per run.

## Scope

In:
- `lym adopt` holds the runs it takes over, and the released tasks not yet started under the adopted subjects bar front tasks, until the subject's adopt check is green.
- One adopt check per adopted subject, run in that subject's worktree.
- A green check lifts its subject's holds and starts the held runs; a failed one files a front darn fix task in the subject.
- A general front-task piece: lym files a task that runs ahead of a worktree's other starts and runs a registered action when it lands, reusable by note `parent-merge-task`.
- The darn writer's stop on a condition the task body names.
- `lym adopt` reports one line per run.
- Dobby gives a held run no pass.
- The help text and package docs of `lym adopt`, the dobby and the board follow.

Out:
- A release verb and a reed button: the parent ruled both out (D3).
- Note `parent-merge-task`'s own command and recipe: it reuses D8's piece in its own task.
- Keeping a run's binary or Bolt version across the move: note `run-pins-its-version`.
- Any change to how the old machine behaves: it releases nothing, and loses ownership when the new one adopts.

## Decisions

### D1 — `lym adopt` is the one takeover verb

The verb the board entry asks for is `lym adopt <beam|root>...` and `lym adopt --all`, reshaped; no second verb is added.
Its steps after this task: origin reconcile (the sync), the local-commit stop with `--keep-local` or `--drop-local` (settling conflicts), the missing pairs added, the holds written (D4), the owner write, the beam steps (dobby and orch through `orchcli.BeamUp`), one adopt check per adopted subject (D5), and the run report (D7).
Every step stays idempotent, so a re-run after a partial adopt finishes it.

Rationale: `lym adopt` already does every step the move needs bar the hold, the fix loop and the report (its help, steps 1 to 8, and `internal/adoptcli/doc.go`); a separate verb would wrap it and duplicate its refusals.

### D2 — The origin reconcile is the sync of Bolt, warp and weft

Adopt's first step, `fabricengine.ReconcileOrigin` over the whole hub in a moving scope, stays the only sync; adopt runs no `lym fabric pull`.
A branch of the prime pair, or of an adopted beam's pair, that the reconcile leaves unmoved because it was judged `rewritten`, `diverged` or `ahead` is reported as the envelope's warning for that worktree, naming its branch, its state and `lym fabric pull` in that worktree, followed by a re-run of the same `lym adopt`; adopt goes on.

Rationale, from the code at this branch's base (`2de97ce7`): `ReconcileOrigin`'s targets are every warp worktree on this machine (the prime, each beam pair and each run pair), each one's records sibling, and Bolt; it fast-forwards each branch behind origin and replays a records branch's local commits.
Adopt today refuses only on a stop of Bolt or the prime's records branch, and stops a slug only when it reads owned elsewhere, so a rewritten or diverged code branch of the prime or an adopted beam passed silently.

### D3 — Parent decisions: the runs wait for the adopt check, and adopt fixes a failed check

Recorded from the parent (operator via `ly:orch`, creel #211 and #212, 2026-11-01); they replace the board entry's bullets that every adopted run stays held until the operator lets it go on and that a verb and a reed button release the hold.
- Asking to take a beam over means its runs shall run here; there is no hold released by hand, no release verb and no reed button.
- The runs wait only for the adopt check and start by themselves when it ends green; when it fails no run starts, and they start once a new check is green.
- When the check fails, lym files a darn task in the beam itself, its body holding the failing tiers and tests and excerpts from `suite.log`, and releases it at once; it is the only run that starts.
- The darn agent diagnoses first: a code failure it fixes, with its perch, and lands as usual; a machine failure (a missing tool, an old version, timing such as OSL-1046's Cortex XDR holding new binaries) has nothing to commit, so it stops and tells the operator exactly what to do.
- When the fix lands the check runs again by itself: green starts the other runs, red files the same darn again (under a new slug, D6), up to a configured number of tries, after which it goes to the operator.
- The mechanism of a darn lym files ahead of a beam's other runs is one general piece note `parent-merge-task` can reuse, not one tied to the adopt check.
- The per-run status lines stay (D7).

### D4 — The hold: a pause entry of a new caller, written before the owner write

Adopt holds, through a slug-scoped entry in the pause record carrying a new fixed caller, `pauserecord.HoldCaller` (new), beside `pauserecord.OwnerCaller`:
- each moved run (as `movedRuns` lists them after the reconcile) not done whose effective owner, read from the Bolt the reconcile left and before the owner write, is another machine, and whose effective owner after this call's owner write will be this machine;
- each task under an adopted subject that is released with an empty status and is not a front task (an entry whose `front` field is true, D8), since dobby's start already waits on a pause covering the task's slug (`internal/dobbyengine/starts.go`); a re-run of `lym adopt` therefore never holds the subject's fix task (D6).

Adopt appends each held slug to its subject's holds file, `holds.json` (new), beside the check record in the subject's directory below `fabricengine.AdoptScratchDir` (D5), written through `internal/state` under its own lock, so the subject's green check lifts exactly those.
The check reads the holds file at its end and never writes it, bar the green end removing the slugs it lifted, in one update under the file's lock, so a slug a second adopt appends during the lift stays listed; the check record's own writes therefore never touch the list.

Rules:
- Adopt writes the holds before its owner write, so a dobby already live here finds the run covered when the owner write lands.
- A run whose own scope holds an owner entry has that entry turned into the hold in the same `pauserecord.Update`, so the run is never uncovered between the two; adopt's lift of owner entries applies only to runs it does not hold.
- A run whose own scope holds an entry of any other caller, such as an operator's `lym tender pause <slug>`, keeps that entry and gets no hold; it reports `paused` (D7).
- The hold is written straight through `pauserecord.Update`, never through `tenderengine.Pause`, so it waits no grace and sends no Esc.
- Not held: a run whose pair `LivePair` reads live here, which reports running here; a run that reads owned elsewhere after the owner write, such as a nested beam keeping its own owner or a beam's pawl run whose parent this machine does not own, which reports owned elsewhere as today.
- Adopt resumes no run itself; the resumes move to the check's green end (D5).
- An owner write that fails, or that another machine's write outruns, leaves the holds standing and reports them; a re-run of the same `lym adopt` finds them in place.

Dobby gives a run covered by a `HoldCaller` entry no pass of any kind, continue passes included, and its owner-pause recheck (`internal/dobbyengine/owned.go`) owes no lift pass for such a run, since no owner entry was lifted.
Batten's first pass refusing a covered run with exit code 78 (`dobbyengine.PassExitCovered`) stays the backstop.

A held run takes a `max_runs` slot: dobby's `slotsTaken` counts every listed run not recorded owned elsewhere, bar a pawl run without its landing hold, held runs included, and `startReady` stops starting once the runs in flight reach `max_runs` (template default 3, `internal/dobbyengine/template.yaml`).
D8 lets a front task start past that limit, so a subject with `max_runs` or more held runs still starts its fix task.

Bound: the hold gates only through the pause record, so a path that ignores pauses (a process under the operator's explicit gate exemption) is not held.
A held run's processes push nothing until the lift, since the network gate holds every covered call, and `tenderengine.Lift` spawns the deferred pushes.
A push child under `OwnerEntryExempt` loses its exemption once its owner entry turns into the hold, and waits for the lift.
`lym tender resume <slug>`, and a bare `lym tender resume`, lift a hold by hand like any entry; the run then goes on without the check, and a child halted at Finalize's owner check stays halted until `lym loom resume` in its worktree.
`lym batten cancel <slug>` lifts the hold of the run it ends, as it lifts every live entry of the slug's own scope (`liftSlugPauses` in `internal/battencli/cancelcmd.go`); the green end then finds that hold gone and lifts nothing for it.
`--keep-local` over a live pair leaves that run unheld, so it goes on without the check; the operator chose to keep that pair.
The pause record is machine-local, so a hold never reaches the other machine.

Rationale: the pause record is the one gate dobby, batten passes and agent work already honour; a distinct caller keeps dobby's owner-pause lift and the hibernate wake from treating the hold as theirs.

### D5 — One adopt check per subject, whose end lifts the holds or files the fix

The adopt check runs once per adopted subject, in that subject's worktree: the prime for `root`, the beam's pair for a beam.
The hidden verb becomes `lym adopt check <subject>`; its record, lock and output files sit under a directory of their own per subject below `fabricengine.AdoptScratchDir`, and `lym beam status` and the prime's status-bar mark show each subject's check, prefixed by its subject.
Adopt starts the check of each subject it recorded an owner for, and of each subject whose holds file (D4) lists a hold still standing, so a re-run of `lym adopt` after a failure starts a new check even when no owner changes.
A hand `lym adopt` resets the subject's fix-try count (D6) before it starts the check; a check started by a fix's landing, or by any `adopt-check` action, finds the count as it stands.

A green end:
- lifts, through `tenderengine.Lift`, each `HoldCaller` entry its subject's holds file lists that still stands, then removes from the holds file the slugs it read there (D4);
- brings up the subject's dobby and orch through `orchcli.BeamUp`, which leaves a live pair alone, so a pair that went down since the adopt drives the runs on;
- starts, for each lifted run, the detached first pass adopt started before this task (`battencli.StartFirstPass`), and `lym loom resume` for a child halted at Finalize's owner check; a start refused because the dobby's own pass at the lift already holds the run's driver lock is no error;
- removes the subject's fix task, if one stands draft, released, running or halted, through `fronttask`'s removal (D8), whose cancel step ends a running one, since a green check needs no fix; the fix task is the subject's front task, open or draft, whose `on_land` is `adopt-check`, found by its `front` field and its project, none for `root`;
- resets the fix-try count (D6).

A failed end, with tiers or tests failing, starts the fix loop (D6).
A check that ends without a verdict, or whose spawn fails, lifts nothing and files nothing; its notice names a re-run of `lym adopt <subject>`.
A start, resume or fix-task removal that fails at the green end is reported in the check's notice to the orch, naming its way forward, and the other runs go on; a failed removal carries the failed step's error and names the hand removal D8 gives for that step.

Bound: a hold written for the subject by a second adopt while its check runs is lifted by that check's green end, though the second adopt's reconcile may have moved the subject's tree under the running check.

Rationale: a fix lands into the subject's own branch (D3), so only a check of that subject's tree can tell the fix worked; the check already reports per tier (`internal/adoptcli/check.go`), and its end is the one point where green or red is known.

### D6 — The fix loop on a failed check

On a failed end, while the subject's fix-try count is below `adopt_fix_tries` (new key in `beam.yaml`, template default 3), the check files the subject's fix task through D8's piece, adding one to the count between D8's create and release writes:
- Slug `<prefix>-t<N>`, where `<prefix>` is `adopt-fix-<subject>` when it leaves room for `-t` and three digits within `boardengine.MaxSlugLength`; for a longer subject, `adopt-fix-`, the subject cut short, a hyphen and the first six hex digits of the full subject's SHA-256, the subject cut to leave the same room, so two subjects sharing a long prefix get two prefixes.
  `<N>` is one more than the highest `N` of a `<prefix>-t<N>` slug found in any of three sources, or 1 when there is none: the board, the run records `shedrun` keeps under `_lym`, and the task branches fabric named for such a slug (its branch prefix followed by the slug), local or on origin.
  The board and the run records reach every machine through the origin reconcile, and a torn-down pair keeps its task branch locally and on origin after `lym batten cleanup --apply` removes its run directory (`doneSlugRefusal` in `internal/battencli/arm.go`), so `<N>` never repeats across a hand adopt's reset of the try count, a machine switch or a cleanup.
  Each filing therefore has a slug that no board entry, run record or task branch of the three sources holds, so the fix task meets neither the done-slug start refusal (`doneSlugRefusal`, which `internal/battencli/dobbyreads.go` sets as `StartRefusal` for a released task whose run status reads done) nor the create row's leftover-branch refusal.
  Kind task, recipe darn, labels `bug` and `gate`, profile `light`, whose darn `code_max_bounces` is 1, so the perch stays on whatever the hub's darn default becomes; in the beam's project for a beam, with no project for `root`.
- Its body holds the failing tiers, the failing tests the gate named, excerpts of `suite.log` around each failure, the instruction to tell a code failure from a machine failure, and D9's stop condition for a machine failure, whose stop report names the exact steps the operator must take and, as its way forward, a re-run of `lym adopt <subject>` once the machine is settled, never `lym loom resume`.
- Its on-land action is `adopt-check`, which starts `lym adopt check <subject>` again.
- The subject's own fix task, its front task in the subject's project whose `on_land` is `adopt-check`, is removed through `fronttask`'s removal before the filing when it stands draft, such as one a hand `lym batten cancel` or a filing that died between its two writes (D8) left, or halted, its entry's run status having the state `blocked` or `failed`, or `awaiting` with no review escalation the arbiter would admit (`EscalationPending` in `internal/sheddriver/driverules.go`).
  The filing does not go ahead while a removal it needed failed; its notice carries the failed step's error and names the hand removal D8 gives for that step, and a re-run of `lym adopt <subject>` only when the failed step came after the record's drop, since a hand removal over a record still standing runs the task's `adopt-check` action itself.
- While the subject's own fix task stands open with the empty status, or with a run status whose state is `running`, `paused`, or `awaiting` with a review escalation the arbiter would admit, the check files none and removes none.
  Its failed notice names that task and its state, says that the task's landing starts the next check, and names the hand step: for `paused`, `lym batten run <slug>` from the prime, which resumes it at the row it halted at; to drop the task, `lym batten cancel <slug> --remove` then `lym board remove <slug>`, whose removal runs the task's `adopt-check` action.
- While any other front task stands open in the subject's worktree (D8), found by its `front` field and its project, never by slug, the check files none, and the failed notice names that task and a re-run of `lym adopt <subject>` once it lands, since a front task of another action starts no check.

The fix-try count lives in the subject's tries file, `fix-tries.json` (new), beside the holds file in the subject's directory below `fabricengine.AdoptScratchDir`, written through `internal/state` under its own lock; the check record's writes never touch it.
A filing adds one to it after D8's create write and before its release write, so a filing that dies between the two still counts and one refused at the create does not.
Only a hand `lym adopt`, writing zero before it starts the check, and a green end (D5) reset it, so the check never needs to tell a hand-started check from one a landing started.

With the count at `adopt_fix_tries`, the check files nothing, and its notice to the orch says the tries ran out and names the failing tiers and tests, `suite.log` and a re-run of `lym adopt <subject>` once the operator has settled it.
A fix task that stops for a machine failure halts as any darn stop does, and the operator, once the machine is settled, re-runs `lym adopt <subject>`, whose check removes the halted task: at its green end (D5), or before filing a new fix task at a red one.

Bound: the loop files at most `adopt_fix_tries` fix tasks per subject between two hand adopts, and at most one at a time.

### D7 — One line per run in adopt's output

Adopt's envelope carries `runs`, one item per run under the named subjects, sorted by slug.
The runs are the union of `movedRuns` before the reconcile, `movedRuns` after it, the released tasks D4 holds and the subject's open front task.
Each item carries its slug, its state, the row it stands at (the batten run record's current producer row, none for an entry with no run record), `owned_by` when owned elsewhere, and a plain `line` such as `dobby-land-notice-needs-flag: waiting for the adopt check at Code-Bouncer`.
The states, with precedence in this order when more than one would apply:
- `owned_elsewhere` — a run whose entry reads owned by another machine after the owner write, with `owned_by`.
- `landed` — a run this machine's records held as not done before the reconcile and whose board entry is absent or reads done after it, since Land's `PruneBoard` calls `boardengine`'s `Prune`, which removes every done entry (`internal/battencli/land.go`, `internal/boardengine/board.go`).
  An entry removed by hand on the other machine reads `landed` too.
- `held_failed_check` — a held run, a released task D4 holds included, while its subject's check record reads failed or ended without a verdict.
- `waiting_for_check` — a held run, a released task D4 holds included, while its subject's check runs or has not started.
- `paused` — a run whose own scope holds an entry of another caller, naming that caller.
- `halted` — a run this machine owns whose batten run record's state is `blocked` or `failed`, or `awaiting`, which behaves like blocked wherever a halt is observed (`internal/shedengine/status.go`), with no review escalation the arbiter would admit, such as a fix task stopped on a machine failure (D9), naming the way forward its stop report gives, for a fix task the re-run of `lym adopt <subject>` (D6).
- `ended` — a run this machine's records held as not done before the reconcile whose entry after it reads a status that is neither empty, a run status nor done, such as the draft a cancel on the other machine writes, and whose run record is gone, naming the entry's status.
- `running_here` — any other run this machine owns, the fix task included, an unstarted fix task with no row.

The per-slug outcomes adopt carries today stay; `runs` is added beside them.
The standing-pause text of a held run names its subject's adopt check and the re-run of `lym adopt`, never `lym tender resume`.

Rationale: the operator and the orch could not tell after the move whether a run had finished, should run or should not; one closed set of states read from the board, the run record, the pause record and the check record answers that.

### D8 — Front tasks: a task lym files ahead of a worktree's other starts

A new package, `internal/fronttask` (new), files a front task: a `boardengine` create of a task carrying two new board fields, `front` (true) and `on_land` (an action key), in the project of the worktree it fronts, which the board's draft rule (`applyDraftRule` in `internal/boardengine/draft.go`) makes draft, followed by a release write setting a null status.
A filing that dies between the two writes leaves a draft front task, which is closed and so holds no start; D6's pre-filing removal or D5's green-end removal at the subject's next check removes it.
For an adopt fix task the filing check died with it, and its record, a running one whose process is gone, shows in `lym beam status` and the status-bar mark as a check that died (`checkView` in `internal/adoptcli/checkview.go`); the operator's re-run of `lym adopt <subject>` starts that next check.
- A front task stands open while its entry carries the empty status or a run status; done, removed or in any other status, such as the draft status `lym batten cancel` writes, it is closed.
- `fronttask` files no front task in a worktree where one stands open, found by its `front` field and its project, none for the prime.
- While a front task stands open, its worktree's dobby starts no other task there; runs already going are not touched, since a caller that needs them stopped holds them itself, as adopt does (D4).
- Dobby starts an open front task even when the runs in flight fill `max_runs`, held runs included (D4); once started it takes its slot like any run.
- `fronttask` keeps a machine-local record of each front task it filed, with its `on_land` action and whether that ran, written after the release write; dobby records a front entry it sees open on the board with no record, reading `on_land` from the entry, so a filing that dies between the release write and the record is healed before dobby starts the task.
  Dobby of the task's parent worktree runs the action once, with the spawn logged, when a recorded task's entry reads done or is gone, since Land prunes the landed entry, and marks it run.
  The record outlives a dobby restart, so a front task that lands while its dobby is down gets its action at the dobby's next tick.
- Dobby records a front entry it sees with no record, and runs a recorded task's action, only while this machine is the effective owner of the fronted worktree's subject (the beam's owner, or the prime owner for the prime), read through `boardengine`'s owner read (`EffectiveOwner`, `PrimeOwner`); a record on a machine that reads the subject owned elsewhere is dropped without running its action.
  `lym adopt check <subject>` makes the same owner test at its start; on a subject owned elsewhere it runs nothing and exits with a notice naming `lym adopt <subject>` on this machine.
- `on_land` names a key of `fronttask`'s closed action table, never a command; the table's one row now is `adopt-check`, which spawns `lym adopt check <subject>` for the subject of the task's parent worktree. Note `parent-merge-task` adds its own row.
- `fronttask` removes a front task in three steps: `lym batten cancel <slug> --remove`, which leaves the entry draft and so closed, run only when the slug has a run claim, record or pair, then the drop of its record, then the board entry's removal; cancel's nothing-left refusal (`refuseNothingLeft` in `internal/battencli/cancel.go`), which a released task that never started or a draft one a hand cancel left meets, counts as the first step done, and any other cancel refusal, such as one when origin cannot be read, fails the step, so the action never runs; a step that fails leaves the steps before it done, and a failed cancel leaves the record, so the task's later removal by hand still runs its action once.
  The hand removal for a failed cancel is `lym batten cancel <slug> --remove` on cancel's own way forward, then `lym board remove <slug>`; for a later failed step it is `lym board remove <slug>`.
  Cancel alone writes the draft status over the run status (`clearRunClaim` in `internal/battencli/cancelcmd.go`) and keeps the pair unless `--remove` is set.
- A front task that halts runs no action.
- `front` and `on_land` are written only through `fronttask`: `boardengine` refuses a value of either that differs from the stored one on every payload write path, `lym board upsert`, `upsert-batch` and `merge`, and drops an equal one, as `checkBeamPayload` does for `beam`, bar `fronttask`'s own write; the refusal names that only lym files a front task and that the payload must drop both keys.
- `boardengine` refuses a change of project or kind on an entry whose stored `front` is true, on the same payload write paths, and `lym board rename` of such an entry; the refusal names that a front task's project, kind and slug are fixed, and as its way forward its removal with `lym board remove <slug>`, preceded by `lym batten cancel <slug> --remove` while a run holds the entry; that hand removal runs the task's `on_land` action, which for an adopt fix task checks the subject again and files a new fix task on red.
- `fronttask`'s `boardengine` writes carry no strand name, so the run-agent task guard (`boardengine`'s `checkTaskCreation`), which judges only `Config.StrandName` and which `boardcli` alone sets from `LYM_STRAND_NAME`, does not judge it.

Bound: a front task holds only new starts in its own worktree, never a nested beam's; its action runs only from the closed table, so a board edit cannot make dobby run an arbitrary command, and its project, kind and slug are fixed, so a board edit cannot move it to another worktree or slug.
A worktree runs at most one front task past `max_runs` at a time, since `fronttask` files none while one stands open.
Only `fronttask` files a task past the run-agent task guard, only a caller of its closed set calls it, the adopt check now, and the parent authorised lym filing the fix task (D3, creel #212).
An entry removed by hand, not through `fronttask`, reads gone and its action runs; for `adopt-check` that is one more check.

Rationale: the parent asked for one general piece (D3); filing, the start hold, the land action and the removal are what both the adopt fix and a merge-in from parent need, and nothing of the adopt check is in it but its table row.

### D9 — The darn writer stops on a condition its task names

The darn writer's stencil gains a second stop: when the task's body names a stop condition and the writer finds it holds, the writer writes its stop report with the content the body asks for and creates the stop marker, ending its turn with no commit, as for an outdated entry today (`darn-template-write.md`'s stop).
The writer marks this stop with a second marker file beside the stop marker, so the darn producer (`internal/loomshed/darnwrite.go`) tells the two stops apart: for a stop on a condition the body names, it puts the stop report's own way forward in the halt reason and appends no `darnStopWayForward`.
The run then halts and the operator is told, as for any darn stop.

Bound: the condition is the task body's own text, so only a task whose body names one gets this stop; a writer never stops this way on a condition it infers.

Rationale: a machine failure has nothing to commit (D3), and the darn stop marker is the existing way a darn writer ends with no commit and a report for the operator.

### Added after Discussion (parent, 2026-11-01): The fix task's gate label only where the board knows it

Decision: failedEnd files the gate label on the fix task only when the hub's board vocabulary knows it, read through boardengine's vocabulary; otherwise it files the task with label bug alone. TestFailedEnd gains the row of a vocabulary without gate, filing the task with label bug alone. Card 13 carries this.

Rationale: Plan-Burler-Fix round 1 finding F3: the board template's vocabulary has no labels, so a hard label gate makes validateLabels refuse every fix filing on such a hub and the fix loop never files; the type label alone is always valid.

## Constraints

- `PATTERN-owner-machine`: the holds sit beside the owner verdict and change no owner read; the owner a hold is decided by is read from the Bolt the reconcile left, and adopt's owner write stays `boardengine`'s owner setter.
- `PATTERN-dobby-orch-pair`: adopt and the check's green end bring a dobby up only through `orchcli.BeamUp`.
- `PATTERN-dobby-starts-runs`: the fix task is started by its parent's dobby from an empty status; the check files it and starts no pass of it.
- `PATTERN-tolerant-records`: `HoldCaller`, `front` and `on_land` are new values and keys an older binary reads past.
- `PATTERN-spawn-observability`: the check's `lym loom resume` spawns and dobby's `on_land` spawn are logged.
- `PATTERN-stencil-ownership`: the fix task's body points at the darn stop by naming its condition, never restating the stop-report format.
- `PATTERN-config-layers`: `adopt_fix_tries` is read through `configengine` with the `beam` module's template.
- `PATTERN-refusal-way-forward`: `boardengine`'s refusal of `front` and `on_land` names its way forward, and a test reaches it through each of `lym board upsert`, `upsert-batch` and `merge`; its refusal of a front task's project, kind or slug change names its way forward, and a test reaches it through `lym board upsert` and `lym board rename`.

## Auto-mode assumptions

- The takeover verb is `lym adopt`, reshaped, not a new verb (D1).
- A released task not yet started under an adopted subject, bar a front task, waits for the check too (D4).
- The check runs per subject, since a fix lands in the subject's own branch (D5).
- `adopt_fix_tries` lives in `beam.yaml`, default 3 (D6).
- A run under an operator's own pause reports `paused`, a state beside the parent's list (D7).
- `landed` covers runs whose board entry turned done or was pruned since this machine last held their record (D7).
- A run cancelled on the other machine reports `ended`, a state beside the parent's list (D7).
- The held slugs live in a holds file of their own per subject, apart from the check record the check rewrites (D4).
- Each fix task's slug carries a number derived from the board, the run records and the task branches, so no filing reuses a slug (D6).
- The fix-try count lives in a tries file of its own per subject, apart from the check record the check rewrites (D6).
- A front task is filed by a create and a release write, as the board's draft rule requires (D8).
- The darn stop on a body's condition is told apart by a second marker file (D9).

## Open risks

None.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - fix-slug-number-lost-after-cleanup (class design, severity MEDIUM, open in rounds 3)
  - fix-try-count-has-no-store (class design, severity MEDIUM, open in rounds 3)
  - red-check-over-live-own-fix-task (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: `lym adopt <beam>` on a machine that does not own the beam reconciles origin, adds the missing pair, writes the holds, writes the owner, runs the beam steps and starts the beam's adopt check, and starts no batten pass; its help lists the new steps.
- D1: a second `lym adopt <beam>` right after the first changes no owner and reports the same states for the runs not done.
- D2: adopt runs no `lym fabric pull`; a run pair, a beam pair and the prime pair behind origin are fast-forwarded by the reconcile step, and a rewritten or diverged code branch of the prime or an adopted beam is a warning naming `lym fabric pull`.
- D3: held runs start by themselves only at a green check of their subject; the only hand lifts are `lym tender resume`, bare or by slug, and `lym batten cancel <slug>`, which lifts the holds of the run it ends; no release verb and no reed button exists.
- D4: a moved run not done under a beam owned elsewhere, read from the reconciled Bolt, gets a `HoldCaller` entry of its own scope, written before the owner write and listed in the subject's holds file; the check's own record writes leave the holds file's list in place, and its green end lifts every hold listed.
- D4: a beam the local Bolt still names as this machine's but origin names as another's gets its runs held.
- D4: a released task with an empty status under an adopted beam gets a hold, and dobby starts it only after the lift.
- D4: a re-run of `lym adopt <subject>` while the subject's fix task stands released with an empty status leaves that task unheld, and dobby starts it unless that re-run's check ends green first, which removes it (D5).
- D4: a moved run whose scope holds an owner entry ends with one live entry of its scope, carrying `HoldCaller`.
- D4: a moved run whose scope holds an operator's entry keeps it and gets no hold; a run whose pair is live here, and a run owned elsewhere after the owner write, get no hold.
- D4: dobby gives a `HoldCaller`-covered run no full pass and no continue pass, and its owner-pause recheck owes no lift pass for it.
- D4: an owner write that fails leaves the holds in place and the envelope reports them.
- D5: `lym adopt root payments` starts two checks, one in the prime and one in the beam's pair, each with its own record.
- D5: a green check lifts only its subject's holds, brings the subject's pair up, starts a first pass per lifted run, resumes a child halted at Finalize's owner check and removes a draft, released, running or halted fix task, a released one that never started included; a start refused for a held driver lock is no error; after the green end no fix-task entry of the subject remains and dobby starts the subject's released tasks.
- D5: a check that ends without a verdict lifts nothing and files nothing; a re-run of `lym adopt` with holds standing starts a new check even when no owner changes.
- D6: a failed check below the try limit files one `adopt-fix-<subject>-t<N>` darn task in the subject, released, with profile `light`, whose body holds the failing tiers, tests and `suite.log` excerpts and the machine-failure stop condition.
- D6: two subjects too long for the slug limit that share their cut prefix get two fix slugs; a failed check with an open front task in its worktree files none; a failed check at the try limit files none and its notice names the operator's re-run.
- D6: a hand `lym adopt` and a green end reset the try count; a check started by a fix's landing does not; the count stands in the subject's tries file through a check's start and end, which rewrite the check record, and a filing adds one to it.
- D6: after a first fix task landed, a second red check files a fix task under a new slug, and dobby starts it; after the landed first fix task's run directory was removed by `lym batten cleanup --apply`, a second red check files a slug with a higher `N`; after a hand `lym adopt` resets the try count, the next fix task's slug is still new.
- D6: a hand `lym adopt` whose check is red while the subject's fix task stands open with the empty status, running, paused, or awaiting with an escalation the arbiter would admit, files none and removes none, and its notice names that task, its state and its hand step.
- D6: a hand `lym adopt` whose check is red while the subject's fix task stands halted removes it and files a new fix task; one whose needed removal fails files none and its notice names the hand removal for the failed step, `lym batten cancel <slug> --remove` first when the cancel failed.
- D6: two fix tasks filed for one subject on two machines, or before and after a hand `lym adopt`, get two slugs.
- D7: adopt's envelope carries `runs` with one item per run and its row, in each state: `owned_elsewhere`, `landed` (its entry done, and its entry pruned), `held_failed_check`, `waiting_for_check`, `paused`, `halted` (a run record in state `blocked`, `failed`, or `awaiting` with no escalation the arbiter would admit, and a fix task stopped on a machine failure, naming its way forward), `ended` (a run cancelled on the other machine) and `running_here`; a released task D4 holds reads `waiting_for_check` with no row, and an unstarted fix task reads `running_here` with no row.
- D8: while a front task under a worktree is open its dobby starts no other task there; once it lands the dobby runs its `on_land` action once; a halted or removed front task runs none, and a draft one lifts the start hold.
- D8: with `max_runs` held runs in a worktree and a front task filed there, dobby starts the front task.
- D8: a front task that lands while its dobby is down gets its `on_land` action once at the dobby's next tick.
- D8: a front task landing while a second machine's dobby of the same worktree is live runs its action on the subject's owner only; `lym adopt check <subject>` on a machine that does not own the subject runs nothing and names `lym adopt <subject>`.
- D8: `fronttask`'s removal of a released front task that never started, and of a draft one a hand cancel left, removes the entry; a filing that dies between its create and its release leaves a draft entry that starts nothing.
- D8: `lym board upsert` changing a front task's project or kind, and `lym board rename` of it, are refused, naming the hand removal; a released front task that never started is removed by `lym board remove <slug>` alone.
- D8: `lym board upsert`, `upsert-batch` and `merge` each refuse a changed `front` or `on_land`, naming that only lym files a front task and that the payload drops both keys, and pass an entry sent back with its stored values; an `on_land` key outside the table runs nothing.
- D9: a darn writer whose task body names a stop condition that holds writes the stop report, the stop marker and the second marker, and commits nothing; the run's halt reason names the body's way forward and not `lym loom resume`.

## PATTERN changes

- `PATTERN-adopt-hold` — adds: On a takeover, `lym adopt` holds it, and each unstarted released task of the subject bar a front task, with a `pauserecord` hold-caller entry; only the green adopt check of the subject lifts it, never dobby; by hand, `lym tender resume` or `lym batten cancel`.
- `PATTERN-front-task` — adds: Only fronttask files a front task and writes `front` and `on_land`; while one is open (empty or run status) dobby starts no other task in its worktree, and runs its `on_land` action from fronttask's closed table, on the subject's owner only; its project, kind and slug are fixed.

## Notes for the plan writer

- The resume and Finalize-child resume code to move out of adopt sits in `internal/adoptcli/adopt.go`; the check's end is `runCheck` in `internal/adoptcli/check.go`.
- `orchcli.BeamUp` is the pair start adopt already calls.
