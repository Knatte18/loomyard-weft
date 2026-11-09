## Goal

Adopt runs as a detached, step-recorded worker that pauses every adopted run and writes the owner before the deploy

Takes: today's adoptcli steps, holds.json hold and green end; testmap-dobby-refresh's landed changes to dobbyengine, beamcli, orchcli and runproc
Delivers: lym adopt as a hidden detached worker the caller follows; the adopt record adopt.json and adopt.lock under fabricengine.AdoptScratchDir with resume and --discard; the D3 step order with the owner write before the deploy; the adopt pause entry replacing the hold and holds.json; --no-orch and its orch-session default; a green check that only reports; the adopt section of lym beam status
Exports: internal/adoptrecord (the record type, its path and a lock-free read, which neither engine import of adoptcli needs), pauserecord.AdoptCaller, the orch-skip field of orchcli.BeamUpOptions, a runproc adopt role

## Scope

The part changes these packages:

- internal/adoptcli (about 1500 changed lines)
- internal/adoptrecord (about 250 changed lines)
- internal/pauserecord (about 60 changed lines)
- internal/tenderengine (about 20 changed lines)
- internal/orchcli (about 40 changed lines)
- internal/dobbyengine (about 60 changed lines)
- internal/beamcli (about 200 changed lines)
- internal/runproc (about 10 changed lines)
- pattern (about 30 changed lines)

## Decisions

### D1 — Adopt runs as a detached worker that the caller follows

`lym adopt` validates its arguments in the caller's process, as `namedEntries` does today, then starts a hidden `lym adopt worker` with the same request.
The worker is started detached in a new session through `internal/proc/`'s detach and `selfexec.LymBinary`, logged through `internal/logger`, reaped by its starter, and registered under `runproc.HubOwner` with an adopt role.
The worker runs the caller's validation again itself and applies D2's rules against the standing record, refusing other subjects and resuming the same ones, before it takes any step.
The worker runs every step of D3 and writes the adopt record (D2); the caller runs no step.
The caller follows the record, printing each step as it ends, and on its end prints the envelope the worker wrote.
The caller exits 0 when the record ends `done`, and non-zero when it ends `stopped` or reads `interrupted`;
an `interrupted` end names the re-run of the same `lym adopt` and `lym adopt --discard`.
The caller knows its worker by pid: once its worker has exited and the record names another pid, its worker lost the lock (D2).
The caller follows another pid's record, whatever its state, only when its start is later than the caller's own start.
It follows that record when its subjects match its own request and otherwise refuses as D2 does.
A caller whose worker has exited with no such record standing reads the record again at its next wake;
with still none, it exits non-zero naming the re-run of the same `lym adopt` and `lym beam status`.
The follow wakes at most once a second and backs off while nothing changes.
In an agent session the follow is capped by the keep-warm resolver, per `PATTERN-daemon-wakeups`;
at the cap it exits non-zero with `still running`, naming `lym beam status` and the re-run of the same `lym adopt` to follow again.
In a plain shell the follow waits until the worker ends.

Rationale: an adopt run from the orch died with the orch's session mid-deploy.
A worker in its own session outlives its caller, and the record carries the result to whoever follows next.
The non-zero exit at the cap and at a stop keeps a shortcut from going on past an adopt that has not ended `done` (D10).

Bound: the worker is a second entry into the adopt steps, and the hidden verb is callable from any shell.
A direct call is one more caller, bounded by the same validation, D2's subject match and `adopt.lock`.
`lym fabric anchor` ends the prime's hub-owner processes through `runproc.EndSparing`, sparing only push and sync, so it ends a worker mid-step;
the record then reads `interrupted`, and D2's resume picks it up.

### D2 — The adopt record and its resume

The worker keeps one adopt record per hub, `adopt.json` directly under `fabricengine.AdoptScratchDir`, beside the subject directories, written through `internal/state` and tolerant of unknown keys.
It holds the request (subjects, `--all`, `--keep-local`, `--drop-local`, `--no-orch`), the resolved subject set, the worker's pid and start ticks, its start,
the steps done with their end times and each one's outcome, the part of the end envelope that step contributes, the step in flight,
the inputs later steps need (moved runs, the projects a local-commit stop held back, the subjects whose owner write stood, the slugs it paused), its state and, for a stop, the error and its way forward.
At `done` or a stop it also holds the end envelope the worker writes, built from the outcomes of every step the record lists as done, earlier processes' steps included.
Its state is `running`, `stopped` (a step refused, or a project held back, with the way forward) or `done`.
A `running` record whose pid is gone, or alive with other start ticks, reads `interrupted`.
The worker takes `adopt.lock` beside it non-blocking and holds it for its whole run; a worker that loses it exits at once and writes nothing.
A fresh worker writes the record `running` at `reconcile`, with the resolved subject set, before any step runs.
A resuming worker rewrites only the pid, start ticks, start and state, `running` at the step it resumes at, keeping the done steps and stored inputs.

When `lym adopt` starts and an unfinished record stands (`running`, `interrupted` or `stopped`):

- With the same request subjects and a live worker, it follows that worker and spawns nothing.
  `--all` matches a record made by `--all`, whatever set either resolves to.
- With the same subjects and no live worker, it starts a worker that resumes.
  A resume whose first step not done is `owner` or earlier starts at `reconcile`, so its owner write always follows a fresh reconcile;
  a resume past `owner` starts at its first step not done.
  The resume keeps the record's flags and adds the call's;
  a call's `--keep-local` or `--drop-local` replaces the record's other one rather than adding to it.
  A call that adds `--keep-local` or `--drop-local` resumes at `reconcile`, since those flags act only there.
  A resumed step reads its inputs from the record, never from the earlier process.
- With other subjects, it refuses, naming the record's subjects, the step it stopped at, the re-run with those subjects, and `lym adopt --discard`.

`lym adopt --discard` takes `adopt.lock` non-blocking and refuses while the lock is held, naming `lym beam status`.
Holding the lock, it removes an unfinished record.
A `done` record is replaced by the next adopt's.

Rationale: every step is safe to run again, so a resume never repeats a step's effect.
Same-subject matching keeps a resume from silently widening or narrowing an adopt.

Bound: `--discard` lifts no pause and undoes no owner write; the steps already done stand.
A resume past `owner` does not re-read origin for the steps before its first step not done; the steps that read origin themselves still do.

### D3 — The step order

The worker runs these steps in order, recording each:

1. `reconcile` — the hub-scope origin reconcile and the local-commit stops with `--keep-local` and `--drop-local`, as today.
2. `pairs` — the pairs this machine lacks, as today.
3. `pause` — the adopt pause of D5.
4. `owner` — the owner write through `boardengine.SetOwners`, before the deploy (D6).
5. `deploy` — the deploy when lym is behind, as today.
6. `beam` — the beam steps of D7, for each subject whose owner write stood.
7. `check` — the start of an adopt check for each subject whose owner write stood, through today's start, which skips a check already running.
8. `done` — the record ends `done`, or `stopped` when a local-commit stop held a project back.

Today's settled-runs step, which lifted the owner entry of each slug adopt did not hold, is dropped: adopt lifts no pause.
An adopted beam's own owner entry, which that step lifted, D5 turns into an adopt pause instead.
A refused step ends the record `stopped` at that step with its way forward, and no later step runs.
A project a local-commit stop holds back is recorded and the other projects go on through every step, as today;
the record then ends `stopped`, naming the re-run with `--keep-local` or `--drop-local`, so D4 still covers the held-back subject.

Rationale: the board's proposed algorithm, with the record (D2) in place of its detach-and-record step and the operator's resume outside adopt.

### D5 — The adopt pause replaces the adopt hold

The `pause` step writes, through one `pauserecord.Update`, an entry of a new caller, `pauserecord.AdoptCaller` (new), with the value `adopt`, for:
each moved run that went through the takeover with no stop and whose owner the write makes this machine's, whether or not its pair reads live here,
and each released, unstarted task of a subject being written that is not a front task.
The step skips every slug the record already lists as paused, so a re-run pauses only slugs it has not paused before.
An existing `OwnerCaller` entry of the slug's own scope is turned into an `AdoptCaller` entry in place, keeping its pause time.
The step turns the existing own-scope `OwnerCaller` entry of each adopted beam's own pair slug into an `AdoptCaller` entry the same way;
it writes no entry for a beam with none, and `root` has no pair slug.
The record lists a converted beam slug among the slugs it paused.
That entry covers the beam pair slug alone, as the `owner` entry did; the beam's runs and tasks carry their own entries.
A slug already holding an entry of another caller keeps it and gets none.
The entry is written straight through the record, as the hold is today: it waits no grace and sends no Esc, so a live strand finishes the step it is in.
The record lists the slugs it paused; `holds.json` is no longer written or read.

The adopt pause is an operator pause, as the board asks: only `lym tender resume` or `lym batten cancel` lifts it.
The adopt check, the dobby's owner lift and a hibernate wake never lift it.
The dobby treats it as it treats the hold today: no pass of any kind and no task start until it is lifted, then the full pass at the lift.
A live `pauserecord.HoldCaller` entry an older adopt left reads the same way and is likewise lifted only by hand.
`lym tender pause` on an adopt-paused slug answers `already_paused: true`, which is now true of the operator's own pause.
The run report's `waiting_for_check` and `held_failed_check` states become one `paused_by_adopt` state, read from a live `adopt` or `adopt-hold` entry of the slug's own scope, whose line names `lym tender resume <slug>`.
A run halted with the owner halt reason also names `lym loom resume` in its pair after the resume.

Rationale: a green check lifted the holds and the dobby started every adopted run, even after the orch paused them.
A run live here under an owner pause would otherwise go on when the dobby's owner lift saw it owned here.
One pause that only the operator lifts makes "adopt never starts work" hold by construction;
the dobby's no-continue-pass treatment answers the board's finding of a `--continue-pass` during a takeover.

Bound: a check started by an older lym, still running across the upgrade, runs the old green end once.
That end can lift the legacy holds it lists, bring up the subject's dobby and orch, start the lifted runs' first passes and resume owner-halted children.

### D6 — The owner write comes before the deploy

The `owner` step runs before the `deploy` step.
A deploy that fails stops the record at `deploy`; the owner stands and the runs D5 paused stay paused, so none of them goes on under the old lym.
An open front task under the subject can start once the owner write stands, before the deploy.

Rationale: the deploy takes minutes, and every minute between the reconcile and the owner write is a minute in which this machine reads a foreign owner.

### D7 — The beam step starts the dobby, and the orch only when asked

The `beam` step starts each subject's dobby through `orchcli.BeamUp` and starts its orch unless the request carries `--no-orch`.
`orchcli.BeamUpOptions` gains a field that skips the orch, set only by adopt.
The caller of D1 sets `--no-orch` on the request when it runs in an orch session, read from its strand name through `agentname.Parse` with role `orch`.
A failed beam step is the subject's warning naming `lym beam start <subject>`, as today; nothing brings it up later, since the green end no longer does (D8).

Rationale: the operator wanted to resume the orch session by hand, and an orch that runs adopt is already up.

Bound: the skip leaves only the orch down; the dobby always starts, and every adopted run is paused (D5), so the dobby starts no adopted work.
`lym beam start` starts the orch later.

### D8 — A green adopt check only reports

The green end no longer lifts pauses, brings up the dobby and orch, starts first passes, or runs `lym loom resume` on owner-halted children.
It keeps removing the subject's fix task and resetting the try count, and posts its notices as today.
The red end is unchanged.

Rationale: the green end is the path that started every adopted run.

### D9 — `lym beam status` shows an unfinished adopt

`lym beam status` adds an `adopt` section when the record is unfinished: its subjects, its state (`running`, `interrupted` or `stopped`), the step in flight or stopped at, the steps done, and the way forward.
The way forward is following the worker for `running`, the re-run of the same `lym adopt` for `interrupted`, and the stop's own way forward for `stopped`, with `lym adopt --discard` for both of the latter.
A `done` record shows nothing.

### Added after Discussion (parent, 2026-11-09): A failed step keeps its gathered part in the end envelope

Decision: Take the review's preferred fix (F2): when a step fails, runSteps stores the part the step gathered before failing in the record (a field such as StoppedPart, cleared when that step later succeeds), and envelopeOf merges it after the done steps without listing the step as done. Card 7's refused-reconcile outcome asserts the envelope keeps the reconcile report and the mutations with partial true; card 6's failed-deploy row keeps asserting that the deploy outcome is named.

Rationale: Today's refusals carry the reconcile report, the mutation record and the deploy outcome; a stopped adopt is exactly when the operator needs them, so the rewrite must not lose them.

## Constraints

- Each step is safe to run again: a re-run adds only missing pairs, pauses only slugs the record does not list as paused, writes the same owners, judges the deploy again, starts beam steps that leave a live pair alone, and starts no second live check.
- The adopt record and `adopt.lock` are never-tracked, under `fabricengine.AdoptScratchDir` (`PATTERN-lym-files`).
  The record is machine-local by nature: it covers only this machine's watcher and dobby.
- Git runs only in `internal/fabricengine` (`PATTERN-fabric-boundary`); the dobby's Bolt pull, the archive and the archive read are fabric calls.

## Auto-mode assumptions

- "No orch start when adopt was run from an orch session" means no orch start for any subject of that adopt.
- An unstarted released task of a subject is an adopted run for D5, since the dobby would otherwise start it.
- The old owner pauses its runs before it archives, the reverse of the board's order, so no pass starts while it archives.

## Open risks

## Acceptance criteria

- D1: `lym adopt` starts a detached worker and runs no step itself; the caller's exit, mid-adopt, leaves the worker running to its end.
- D1: in an agent session the follow ends non-zero with `still running` at the keep-warm cap, naming `lym beam status` and the re-run.
- D1: of two concurrent adopts with other subjects, the caller whose worker lost the lock refuses naming the running adopt's subjects.
- D1: the caller exits 0 for a record that ends `done` and non-zero for one that ends `stopped`; an `interrupted` end names the re-run and `lym adopt --discard`.
- D1: a caller whose worker lost the lock to a resuming worker follows that worker's record, and one whose worker lost it to `--discard` exits non-zero naming the re-run.
- D1: a direct `lym adopt worker` call with other subjects than an unfinished record's is refused, and one with invalid arguments is refused before any step.
- D2: a worker killed after the `owner` step, then a re-run with the same subjects, resumes at `deploy` and runs no reconcile, pair add, pause or owner write again.
- D2: a worker killed after the `pairs` step, then a re-run with the same subjects, re-runs the reconcile before its owner write.
- D2: a re-run while a live worker runs follows it and spawns nothing; a re-run with other subjects is refused naming the recorded subjects and `lym adopt --discard`.
- D2: a resume keeps the record's `--no-orch`; a resume adding `--keep-local` restarts at `reconcile`, and one adding `--drop-local` to a record with `--keep-local` carries `--drop-local` alone.
- D2: `lym adopt --discard` removes an interrupted record; a discard while a worker holds `adopt.lock` is refused and leaves its record standing, and a worker that meets a discard holding the lock exits writing nothing, its caller exiting non-zero naming the re-run.
- D3: the steps run in the order D3 lists, and a refused step ends the record `stopped` at that step with no later step run.
- D3: a project held back by a local-commit stop ends the record `stopped` while the other subjects reach `check`.
- D5: each adopted run, live here or not, and each released task that is not a front task gets an `adopt` pause entry; an `owner` entry is turned into one in place, keeping its pause time, and a slug already holding an entry of any caller but `owner` keeps it and gets none.
- D5: an adopted beam's own-scope `owner` entry on its pair slug is turned into an `adopt` entry in place, which `lym tender resume <beam>` lifts and the run report names; a beam with no such entry gets none.
- D5: a re-run of the `pause` step leaves a slug the record lists as paused, and the operator has resumed since, unpaused.
- D5: a green adopt check, the dobby's owner lift and a hibernate wake leave every adopt pause and legacy `adopt-hold` entry standing; `lym tender resume <slug>` lifts it, and the dobby then starts the full pass.
- D5: `lym tender pause` on an adopt-paused slug answers `already_paused: true`, and the run report shows `paused_by_adopt` naming `lym tender resume <slug>`.
- D6: the owner write runs before the deploy; a failed deploy leaves the owner written, the runs paused and the record stopped at `deploy`.
- D7: with `--no-orch`, and from an orch strand, the beam step starts the dobby and no orch; without either it starts both.
- D8: the green end starts no pass, brings up no dobby or orch and runs no `lym loom resume`; it still removes the fix task and resets the try count.
- D9: `lym beam status` shows the adopt section for a running, interrupted and stopped record, naming the step and the way forward, and shows none for a done record.
- Review: `internal/adoptcli/doc.go` and the `PATTERN-dobby-runs` background describe the worker, the record, the step order and the adopt pause, and no longer say a green check lifts holds or starts runs.
  The background's Dobby and orch pair rule names the adopt beam step's dobby-only start under `--no-orch`.

## PATTERN changes

- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt pause lifts only by hand.
