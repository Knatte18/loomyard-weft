# Decision record — machine-switch

## Goal

Taking a beam, or root, over onto this machine is one verb, `lym adopt`, that leaves every run under it in a known state.
It syncs, adopts, settles conflicts and starts each beam's dobby and orch.
The runs it takes over run here once the adopt check of their subject is green: they wait for the check and start by themselves on green.
A failed check files a darn fix task in that subject, the only run that starts; when the fix lands the check runs again, up to a configured number of tries, then the failure goes to the operator.
It prints one line per run.

## Scope

In:
- `lym adopt` holds the runs it takes over, and the released tasks not yet started under the adopted subjects, until the subject's adopt check is green.
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
- When the fix lands the check runs again by itself: green starts the other runs, red files the same darn again, up to a configured number of tries, after which it goes to the operator.
- The mechanism of a darn lym files ahead of a beam's other runs is one general piece note `parent-merge-task` can reuse, not one tied to the adopt check.
- The per-run status lines stay (D7).

### D4 — The hold: a pause entry of a new caller, written before the owner write

Adopt holds, through a slug-scoped entry in the pause record carrying a new fixed caller, `pauserecord.HoldCaller` (new), beside `pauserecord.OwnerCaller`:
- each moved run (as `movedRuns` lists them after the reconcile) not done whose effective owner, read from the Bolt the reconcile left and before the owner write, is another machine, and whose effective owner after this call's owner write will be this machine;
- each task under an adopted subject that is released with an empty status, since dobby's start already waits on a pause covering the task's slug (`internal/dobbyengine/starts.go`).

Adopt records each held slug in its subject's check record (D5), so the subject's green check lifts exactly those.

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

Bound: the hold gates only through the pause record, so a path that ignores pauses (a process under the operator's explicit gate exemption) is not held.
A held run's processes push nothing until the lift, since the network gate holds every covered call, and `tenderengine.Lift` spawns the deferred pushes.
A push child under `OwnerEntryExempt` loses its exemption once its owner entry turns into the hold, and waits for the lift.
`lym tender resume <slug>`, and a bare `lym tender resume`, lift a hold by hand like any entry; the run then goes on without the check, and a child halted at Finalize's owner check stays halted until `lym loom resume` in its worktree.
`--keep-local` over a live pair leaves that run unheld, so it goes on without the check; the operator chose to keep that pair.
The pause record is machine-local, so a hold never reaches the other machine.

Rationale: the pause record is the one gate dobby, batten passes and agent work already honour; a distinct caller keeps dobby's owner-pause lift and the hibernate wake from treating the hold as theirs.

### D5 — One adopt check per subject, whose end lifts the holds or files the fix

The adopt check runs once per adopted subject, in that subject's worktree: the prime for `root`, the beam's pair for a beam.
The hidden verb becomes `lym adopt check <subject>`; its record, lock and output files sit under a directory of their own per subject below `fabricengine.AdoptScratchDir`, and `lym beam status` and the prime's status-bar mark show each subject's check, prefixed by its subject.
Adopt starts the check of each subject it recorded an owner for, and of each subject whose record lists a hold still standing, so a re-run of `lym adopt` after a failure starts a new check even when no owner changes.
A hand `lym adopt` resets the subject's fix-try count (D6); a check started by a fix's landing does not.

A green end:
- lifts, through `tenderengine.Lift`, each `HoldCaller` entry its record lists that still stands;
- brings up the subject's dobby and orch through `orchcli.BeamUp`, which leaves a live pair alone, so a pair that went down since the adopt drives the runs on;
- starts, for each lifted run, the detached first pass adopt started before this task (`battencli.StartFirstPass`), and `lym loom resume` for a child halted at Finalize's owner check; a start refused because the dobby's own pass at the lift already holds the run's driver lock is no error;
- cancels the subject's open fix task, if one stands halted, through batten's cancel, since a green check needs no fix;
- resets the fix-try count.

A failed end, with tiers or tests failing, starts the fix loop (D6).
A check that ends without a verdict, or whose spawn fails, lifts nothing and files nothing; its notice names a re-run of `lym adopt <subject>`.
A start or resume that fails at the green end is reported in the check's notice to the orch, and the other runs go on.

Bound: a hold written for the subject by a second adopt while its check runs is lifted by that check's green end, though the second adopt's reconcile may have moved the subject's tree under the running check.

Rationale: a fix lands into the subject's own branch (D3), so only a check of that subject's tree can tell the fix worked; the check already reports per tier (`internal/adoptcli/check.go`), and its end is the one point where green or red is known.

### D6 — The fix loop on a failed check

On a failed end, while the subject's fix-try count is below `adopt_fix_tries` (new key in `beam.yaml`, template default 3), the check files the subject's fix task through D8's piece and adds one to the count:
- Slug `adopt-fix-<subject>`, the subject cut to fit the board's slug limit; kind task, recipe darn, labels `bug` and `gate`; in the beam's project for a beam, with no project for `root`; no profile, so the hub's darn default applies, which keeps the perch on (`light` today, `code_max_bounces` 1).
- Its body holds the failing tiers, the failing tests the gate named, excerpts of `suite.log` around each failure, the instruction to tell a code failure from a machine failure, and D9's stop condition for a machine failure with the exact steps the operator must take.
- Its on-land action is `adopt-check`, which starts `lym adopt check <subject>` again.
- A fix task of the subject that is still open is never filed twice; the failed notice names it instead.

With the count at `adopt_fix_tries`, the check files nothing, and its notice to the orch says the tries ran out and names the failing tiers and tests, `suite.log` and a re-run of `lym adopt <subject>` once the operator has settled it.
A fix task that stops for a machine failure halts as any darn stop does, and the operator, once the machine is settled, re-runs `lym adopt <subject>`, whose green check cancels the halted task (D5).

Bound: the loop files at most `adopt_fix_tries` fix tasks per subject between two hand adopts, and at most one at a time.

### D7 — One line per run in adopt's output

Adopt's envelope carries `runs`, one item per run under the named subjects, sorted by slug, each with its slug, its state, the row it stands at (the batten run record's current producer row), `owned_by` when owned elsewhere, and a plain `line` such as `dobby-land-notice-needs-flag: waiting for the adopt check at Code-Bouncer`.
The states, with precedence in this order when more than one would apply:
- `owned_elsewhere` — a run whose entry reads owned by another machine after the owner write, with `owned_by`.
- `landed` — a run this machine's records held as not done before the reconcile and whose board entry reads done after it.
- `held_failed_check` — a held run while its subject's check record reads failed or ended without a verdict.
- `waiting_for_check` — a held run while its subject's check runs or has not started.
- `paused` — a run whose own scope holds an entry of another caller, naming that caller.
- `running_here` — any other run this machine owns, the fix task included.

The per-slug outcomes adopt carries today stay; `runs` is added beside them.
The standing-pause text of a held run names its subject's adopt check and the re-run of `lym adopt`, never `lym tender resume`.

Rationale: the operator and the orch could not tell after the move whether a run had finished, should run or should not; one closed set of states read from the board, the run record, the pause record and the check record answers that.

### D8 — Front tasks: a task lym files ahead of a worktree's other starts

A new package, `internal/fronttask` (new), files a front task: one `boardengine` write of a released task carrying two new board fields, `front` (true) and `on_land` (an action key), in the project of the worktree it fronts.
- While a front task under a worktree is not done, that worktree's dobby starts no other task under it; runs already going are not touched, since a caller that needs them stopped holds them itself, as adopt does (D4).
- When a front task lands, the dobby of its parent worktree runs its `on_land` action once, with the spawn logged; dobby remembers each front task's action ahead of its Land row, as it does `notify_on_land`, since Land prunes the landed entry.
- `on_land` names a key of `fronttask`'s closed action table, never a command; the table's one row now is `adopt-check`, which spawns `lym adopt check <subject>` for the subject of the task's parent worktree. Note `parent-merge-task` adds its own row.
- `front` and `on_land` are written only through `fronttask`; `lym board upsert` refuses them, naming that only lym files a front task, as it refuses `beam` outside the `lym beam` verbs.
- A front task that halts or is cancelled runs no action, and dobby's start hold lifts once it is done or removed.

Bound: a front task holds only new starts in its own worktree, never a nested beam's; its action runs only from the closed table, so a board edit cannot make dobby run an arbitrary command.

Rationale: the parent asked for one general piece (D3); filing, the start hold and the land action are the three things both the adopt fix and a merge-in from parent need, and nothing of the adopt check is in it but its table row.

### D9 — The darn writer stops on a condition its task names

The darn writer's stencil gains a second stop: when the task's body names a stop condition and the writer finds it holds, the writer writes its stop report with the content the body asks for and creates the stop marker, ending its turn with no commit, as for an outdated entry today (`darn-template-write.md`'s stop).
The run then halts and the operator is told, as for any darn stop.

Bound: the condition is the task body's own text, so only a task whose body names one gets this stop; a writer never stops this way on a condition it infers.

Rationale: a machine failure has nothing to commit (D3), and the darn stop marker is the existing way a darn writer ends with no commit and a report for the operator.

## Constraints

- `PATTERN-owner-machine`: the holds sit beside the owner verdict and change no owner read; the owner a hold is decided by is read from the Bolt the reconcile left, and adopt's owner write stays `boardengine`'s owner setter.
- `PATTERN-dobby-orch-pair`: adopt and the check's green end bring a dobby up only through `orchcli.BeamUp`.
- `PATTERN-dobby-starts-runs`: the fix task is started by its parent's dobby from an empty status; the check files it and starts no pass of it.
- `PATTERN-tolerant-records`: `HoldCaller`, `front` and `on_land` are new values and keys an older binary reads past.
- `PATTERN-spawn-observability`: the check's `lym loom resume` spawns and dobby's `on_land` spawn are logged.
- `PATTERN-stencil-ownership`: the fix task's body points at the darn stop by naming its condition, never restating the stop-report format.
- `PATTERN-config-layers`: `adopt_fix_tries` is read through `configengine` with the `beam` module's template.
- `PATTERN-refusal-way-forward`: `lym board upsert`'s refusal of `front` and `on_land` names its way forward, and a test reaches it.

## Auto-mode assumptions

- The takeover verb is `lym adopt`, reshaped, not a new verb (D1).
- A released task not yet started under an adopted subject waits for the check too (D4).
- The check runs per subject, since a fix lands in the subject's own branch (D5).
- `adopt_fix_tries` lives in `beam.yaml`, default 3 (D6).
- A run under an operator's own pause reports `paused`, a state beside the parent's list (D7).
- `landed` covers runs whose board entry turned done since this machine last held their record (D7).

## Open risks

None.

## Acceptance criteria

- D1: `lym adopt <beam>` on a machine that does not own the beam reconciles origin, adds the missing pair, writes the holds, writes the owner, runs the beam steps and starts the beam's adopt check, and starts no batten pass; its help lists the new steps.
- D1: a second `lym adopt <beam>` right after the first changes no owner and reports the same states for the runs not done.
- D2: adopt runs no `lym fabric pull`; a run pair, a beam pair and the prime pair behind origin are fast-forwarded by the reconcile step, and a rewritten or diverged code branch of the prime or an adopted beam is a warning naming `lym fabric pull`.
- D3: no verb releases a hold by hand other than `lym tender resume`; held runs start only at a green check of their subject.
- D4: a moved run not done under a beam owned elsewhere, read from the reconciled Bolt, gets a `HoldCaller` entry of its own scope, written before the owner write and listed in the subject's check record.
- D4: a beam the local Bolt still names as this machine's but origin names as another's gets its runs held.
- D4: a released task with an empty status under an adopted beam gets a hold, and dobby starts it only after the lift.
- D4: a moved run whose scope holds an owner entry ends with one live entry of its scope, carrying `HoldCaller`.
- D4: a moved run whose scope holds an operator's entry keeps it and gets no hold; a run whose pair is live here, and a run owned elsewhere after the owner write, get no hold.
- D4: dobby gives a `HoldCaller`-covered run no full pass and no continue pass, and its owner-pause recheck owes no lift pass for it.
- D4: an owner write that fails leaves the holds in place and the envelope reports them.
- D5: `lym adopt root payments` starts two checks, one in the prime and one in the beam's pair, each with its own record.
- D5: a green check lifts only its subject's holds, brings the subject's pair up, starts a first pass per lifted run, resumes a child halted at Finalize's owner check and cancels a halted fix task; a start refused for a held driver lock is no error.
- D5: a check that ends without a verdict lifts nothing and files nothing; a re-run of `lym adopt` with holds standing starts a new check even when no owner changes.
- D6: a failed check below the try limit files one `adopt-fix-<subject>` darn task in the subject, released, whose body holds the failing tiers, tests and `suite.log` excerpts and the machine-failure stop condition.
- D6: a failed check with an open fix task files none; a failed check at the try limit files none and its notice names the operator's re-run.
- D6: a hand `lym adopt` resets the try count; a check started by a fix's landing does not.
- D7: adopt's envelope carries `runs` with one item per run and its row, in each state: `owned_elsewhere`, `landed`, `held_failed_check`, `waiting_for_check`, `paused` and `running_here`.
- D8: while a front task under a worktree is open its dobby starts no other task there; once it lands the dobby runs its `on_land` action once; a halted or cancelled front task runs none.
- D8: `lym board upsert` refuses `front` and `on_land`, naming that only lym files a front task; an `on_land` key outside the table runs nothing.
- D9: a darn writer whose task body names a stop condition that holds writes the stop report and the stop marker and commits nothing.

## PATTERN changes

- `PATTERN-adopt-hold` — adds: Taking a run over from another machine: `lym adopt` holds it, and each released task not started under the subject, with a pause entry of `pauserecord`'s hold caller; only its subject's green adopt check lifts it automatically, never dobby; `lym tender resume` lifts it by hand.
- `PATTERN-front-task` — adds: Filing a task lym runs ahead of a worktree's other starts: only the fronttask package, the sole writer of the `front` and `on_land` board fields; dobby starts no other task there while it is open, and runs its `on_land` action only from fronttask's closed table.

## Notes for the plan writer

- The resume and Finalize-child resume code to move out of adopt sits in `internal/adoptcli/adopt.go`; the check's end is `runCheck` in `internal/adoptcli/check.go`.
- Dobby's `notify_on_land` memory in `internal/dobbyengine/landnotice.go` is the model for remembering a front task's action past Land.
- `orchcli.BeamUp` is the pair start adopt already calls.
