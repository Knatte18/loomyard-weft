# Decision record — beam-start-and-ownership

## Goal

A beam, the prime included, is one unit of two parts that run together on the machine that owns it: its dobby and its orch.
`lyx beam start` brings a beam up, ownership sits on projects instead of tasks, `lyx adopt` moves a project, and a beam's life is one batten run over its project entry with the `pawl` recipe.

## Scope

In:

- `lyx beam start [slug] [--all]`, `lyx beam stop [slug]`, `lyx beam status` showing dobby and orch.
- Removal of `lyx orch start` (with `--continue`, `--handoff`, `--adopt`), `lyx orch cleanup`, the `lyx prime` shortcut and `lyx beam promote`; the shortcut `lyx s`.
- Ownership on projects: the root project's owner (board key `prime_owner`) and each beam project's owner; no task owner.
- `lyx adopt <project>...` and `lyx adopt root`, replacing `lyx adopt <task>` and `lyx adopt --prime`.
- The `pawl` recipe, today's `beam` recipe renamed and grown to Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize and Beam-Down.
- `lyx beam create` starting the pawl run, `lyx beam land` ending Beam-Await, batten's start guard, cancel and Worktree-Create for a beam project, the "beam can land" notice.
- Every refusal, notice, stencil, skill and doc that names a removed verb or a task owner.

Out:

- The beam window itself (`beam: <slug>`, `beam: prime`): landed in afd030a4; `lyx beam start` only ensures it as `lyx orch start` does today.
- Loom's and darn's Finalize reroute flags (note `loom-finalize-merges-itself`).
- The adopt machine check's gate key and visibility (note `adopt-check-visible`); adopt's step 6 is unchanged.
- The README's per-project layout (note `readme-per-project-files`).
- Turning a running task into a beam (notes `beam-promote-keeps-discussion`, `loom-beam-darns`).

## Decisions

### D1 — `lyx beam start [slug] [--all]` brings a beam up

`lyx beam start` with no slug acts on the acting worktree's beam (the prime or a marked beam pair); with a slug, on that beam from anywhere in the hub, where the slug `root` names the prime.
In order it: reconciles the hub's config after a binary change through `hubreconcile.Ensure`; runs the hub-scope origin reconcile from the prime; claims the prime when the board records no root owner; pauses each owned_elsewhere run of the report; then runs the beam steps: starts the beam's dobby through `dobbyengine.EnsureLive` for that worktree only, ensures the beam window, starts the orch and its watcher.
These are today's `lyx orch start` steps, moved, with the dobby start narrowed from every owned beam to the one beam.
`--all` runs the same for the prime and every local beam pair this machine owns, one envelope entry per beam, a failure in one never stopping the others.
"The beam steps" below names the dobby start, the window and the orch start alone.

### D2 — The orch at start: resume, else note, else stencil

A live orch strand and a live watcher are left alone; a live strand with a dead watcher gets a watcher; a missing orch is started.
A started orch resumes the session the orch state records; with none recorded it starts fresh from the last completed handoff note; with no note, from the start stencil.
A recorded session whose resume does not come up live, or whose provider transcript is gone, falls back to the fresh start from the note, never to an error, and the envelope names the session it gave up.
A missing dobby is started on its own; the two parts are judged separately.
A fresh orch comes only from `lyx orch refresh` on a live orch, since `lyx orch stop` keeps the recorded session id and the next start resumes it.
Bound: the orch resumes the context the recorded id names; after a reed server rebirth in clear mode that is the pre-clear context, as `--continue` resumes it today.

### D3 — `lyx beam start` refusals

It refuses, before it writes anything:

- outside a hub, naming the hub;
- in a worktree that is no beam (a task pair), naming the beam it belongs to (its parent through `hubgeom.ParentOf`) and that beam's path, and `lyx beam start` there;
- a slug that names no local beam pair, naming `lyx beam create <slug>` from its parent worktree, or `lyx adopt <slug>` when its branches are on origin;
- a beam another machine owns, read from the Bolt the reconcile just pulled, naming the owner and `lyx adopt <slug>` (`lyx adopt root` for the prime);
- a beam with an anchor move standing, naming the move's way forward, as `EnsureLive` does today;
- a beam whose pawl run is past Beam-Await (D9), naming `lyx batten status <slug>`; this replaces today's refusal of a beam whose entry its landing run holds.

### D4 — `lyx s`, `lyx beam stop`, `lyx beam status`, and what `lyx orch` keeps

- `shortcuts.yaml` gains `s: [[beam, start], [reed, attach]]` and drops `prime`.
  `lyx s` takes no slug: every word goes to `reed attach` as the shortcut routing does today, so it acts on the acting worktree's beam.
  A refusal exits non-zero, attaches nothing and leaves its envelope on the terminal; on success the attach covers the output.
- `lyx beam stop [slug]` stops the beam's orch strand and watcher through orch's stop function, then its dobby, each only when live; idempotent.
  The dobby is ended through `runproc` by its `RoleDobby` record alone, never by ending every process of the beam's slug, which would end the pawl run's driver and agents too; `runproc` gains that role-scoped end if it lacks one.
  Bound: a stopped dobby does not drive the beam's runs, and passes already spawned finish their one pass; a hand `lyx batten run` there restarts the dobby, as it does today.
- `lyx beam status` prints, per beam, the dobby section it prints today and the beam's orch: strand live or not, watcher live or not, recorded session id.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw`, `rules` and the hidden `watch` and `resume-context`.
  `lyx orch start` and `lyx orch cleanup` are removed; `lyx cleanup` already removes old handoffs.
  The `--adopt` launch stencil (`orch-template-adopt.md`) goes with `--adopt`.
- Every way forward, notice, stencil and skill naming `lyx orch start` names `lyx beam start`, and every one naming `lyx prime` names `lyx s`.

### D5 — Ownership sits on projects

- The root project's owner is the board's `prime_owner` key, kept under that name in `board.json`; a beam project's owner is its `owner_machine`.
- A task and a grouping project record no owner.
  Their effective owner is the owner of `hubgeom.ParentOf`'s answer: the nearest beam they belong to, or root.
  This narrows the entry's "each project is owned by one machine" to beam projects and root: a grouping project has no worktree and no dobby to drive it, so its tasks are driven by the beam or prime they fork from.
- `Owners.Effective`: root reads `prime_owner`; a beam project reads its recorded owner, else its parent's; any other entry reads its parent's owner.
  A board with no root owner reads a recorded beam owner as recorded and every other entry as owned by no other machine.
  The run-held rule (a run-held entry with no owner reads as no other machine) goes, from `Effective` and from every reader that applies it on its own, such as tender's owner-pause selection and adopt's test of a started entry, which reads the run status.
- A task's or grouping project's `owner_machine` is dropped on load, the way `migrate.go` drops `tier` and `type`, so it leaves `board.json` at the first write after the upgrade and is never read.
- `boardengine`'s owner setter writes only the root owner and beam project owners; its beam write sets the beam flag and `recipe: pawl` in the same commit.
  Batten's claim writes the run status alone and refuses a task whose effective owner is another machine.
- The list and get views carry the effective owner as today; the README's Running Owner column shows the effective owner.
- Upgrade: a run in flight whose recorded task owner differs from its project's owner reads its project's owner after the upgrade, so the machine driving it owner-pauses it at its next pass, naming `lyx adopt <project>`.
  Bound: every machine of the hub runs the same build, deployed through `update-plugins.sh`; until the other machine is updated it still reads and writes task owners, which the new build drops again on its next load.
Rationale: a task follows its project, whose owner's dobby starts and drives it.

### D6 — `lyx adopt <project>...` and `lyx adopt root`

- The arguments are beam project slugs and `root`; `--all` names root and every open beam project; `--keep-local` and `--drop-local` stay.
  `--prime` is removed.
- Refused before anything runs: a task, naming `lyx adopt <its beam or root>`; a grouping project, naming `lyx adopt <its beam or root>`; a note and an unknown slug as today; a project that is no beam, naming `lyx beam create <slug>`.
- An adopt moves the named project alone: its owner, its pair (added when absent and its branches are on origin) and the runs whose parent is its worktree.
  A nested beam keeps its own owner, so `lyx adopt root` moves root's tasks, its grouping projects' tasks and their runs, and never a beam below it.
- The refusals that a parent or a member reads owned by another machine go: no task has an owner, and a beam may be owned apart from its parent.
- After the owner write, adopt resumes each run in flight whose parent is an adopted worktree, as a detached first pass, so a beam's own pawl run is resumed only by adopting its parent.
  A run whose entry reads owned by another machine, such as a nested beam's pawl run under an adopted root, is not resumed and is reported as owned elsewhere, never as an error.
- Adopt then runs each adopted project's beam steps (D1), since adopt's own step 1 already reconciled origin.
- The origin reconcile, `--keep-local`/`--drop-local`, the anchor-move refusal, the owner pause lift and the machine check (step 6) are unchanged.

### D7 — A beam's two parts leave a machine that no longer owns it

A beam's dobby already exits when it reads its beam owned by another machine.
The orch's watcher in a beam or the prime reads the beam's owner from the local Bolt on its tick, and when another machine owns it stops the orch strand and exits, logging the owner.
Rationale: the entry's "both run on the machine that owns the beam"; after `lyx adopt G` elsewhere, G's orch on the old machine would otherwise keep running beside the new one.
Bound: the verdict is as fresh as this machine's local Bolt, and it reaches a watcher started by an older build only once that watcher restarts; until then the orch stays up, and `lyx beam status` shows it.

### D8 — The `pawl` recipe

`contracts/recipes/beam-recipe.yaml` becomes `pawl-recipe.yaml` and `shedrun.RecipeBeam` becomes `RecipePawl` with the value `pawl`.
A seed or status that records the recipe `beam` reads as `pawl`, so a landing run in flight at the upgrade resumes at its row; Beam-Describe and Finalize keep their names.
Rows, in order:

| Row | Does |
|---|---|
| Beam-Up | marks the pair a beam (`fabricengine.MarkBeam`), spawns its push, regenerates the hub workspace, removes any land request, then runs the beam steps (D1), with no reconcile; every step skipped when already done |
| Beam-Await | holds the run until a land request it accepts (D9) |
| Parent-Align | loom's row: merges the parent's upstream tip into the beam, merge fixer on a conflict or failing tree |
| Code-Bouncer, Code-Burler-Review, Code-Burler-Fix | loom's Code-Review perch over the beam's diff, reading loom.yaml's code_* keys as darn does; answer key the project entry snapshot (D9) |
| Beam-Describe | writes the landing description from the entry and the commits, as today |
| Finalize | `landingshed.Finalize` under the parent lock; `ConflictReroutes` false, `LargeMergeLands` true, no `on_reroute` |
| Beam-Down | stops the beam's orch and watcher and, when still live, its dobby, as `lyx beam stop` does; a failure warns and is Done, since Worktree-Teardown ends what is left |

`profiles.yaml`'s template gains a `pawl` block per profile, carrying the perch's settings.

### D9 — Beam-Await and the beam's state after it

- Beam-Await waits in its own driver process, waking on the land request file and on a timer at most once a second with backoff to a minute while idle (`PATTERN-daemon-wakeups`); it spawns no agent.
- A land request names the pawl run it is for, by the run's seed id; Beam-Await accepts only a request naming its own run and removes any other.
- On an accepted request it re-runs `lyx beam land`'s checks (D10).
  When they pass it writes the project entry snapshot `_lyx/discussion/board-entry.md`, the perch's answer key, removes the request, stops the beam's dobby so it starts no new member, and is Done.
  When they fail it removes the request, posts the failure to the beam's own orch and keeps waiting.
- When it reads its beam owned by another machine it removes any request and ends its driver with a yield, deciding nothing; the batten pass's owner check owner-pauses the run.
  Bound: until the owner pause stands, a batten pass may re-spawn the driver once per probe pass, each yielding at once.
- Batten's InnerRun sends no quiet and no driver-dead notice for a child at Beam-Await, as it sends none while a parent lock wait record stands; a restarted batten re-spawns the driver, which re-enters the row.
- While the pawl run is at Beam-Await, its run status is no landing hold: members fork from the beam (`beamForkPrecondition`), every verb runs inside the beam (`ResolveBattenPrime`'s landing refusal), the beam starts (D3), the entry stays in the board's Project layer, and the run lock leaves the project entry's fields editable.
  Once the run is past Beam-Await each of these refuses or locks as today's landing hold does, so a member released after Beam-Await is refused at its Worktree-Create.

### D10 — `lyx beam create` and `lyx beam land`

- `lyx beam create <slug> [--anchor <dir>]` runs from the beam's parent worktree, the prime or the parent beam, and refuses elsewhere naming that worktree's path.
  It takes an open project entry, refuses a note, a task and a done project as today, refuses a beam another machine owns naming `lyx adopt <slug>`, and refuses when the parent reads owned by another machine, naming `lyx adopt <parent>`.
  It claims the root for this machine when the board records no root owner, records through the owner setter this machine as the beam's owner with the beam flag and `recipe: pawl`, then starts the batten run as a detached first pass, driven on by the parent's dobby, and names `lyx batten status <slug>`.
  `--anchor` rides into batten's own run record at that first pass, and Worktree-Create passes it to `Topology.Add`, so a resume keeps it.
  Re-run, it skips what is done: a pawl run in flight is reported live; a flagged beam with no run status, left by a first pass a covering pause refused or that died, gets its first pass again; a legacy beam with a marked pair and no pawl run gets its pawl run, Worktree-Create finding the pair complete and Beam-Up finding it marked.
  Bound: a first pass refused for a covering pause leaves no run, the envelope names the pause, and the same `lyx beam create <slug> [--anchor <dir>]` after the lift starts it; dobby never starts a beam project.
- `lyx beam land <slug>` runs anywhere in the hub and writes a land request naming the pawl run in the beam worktree's `.lyx/beam/`, which Beam-Await consumes.
  It refuses: while any open entry's `hubgeom.ParentOf` is the beam, or any local pair's origin record names the beam as its parent worktree, naming each, as `BeamLandRefusal` does today; a beam this machine does not own, naming `lyx adopt <slug>`; a beam whose parent another machine owns, naming `lyx adopt <parent>` (`lyx adopt root` for the prime); a beam whose pawl run is not running at Beam-Await, naming `lyx batten status <slug>`; a legacy beam with no pawl run, naming `lyx beam create <slug>`.
  The refusal of a live orch goes, since Beam-Down stops it.
- Today's separate landing run of `lyx beam land` goes.
- The board accepts `recipe: pawl` on a project only, and refuses it on a task; a beam project with no recipe reads as pawl.

### D11 — Batten for a beam project

- Batten's start guard lets a beam project start or resume: a fresh start gets a task's dirty-source and left-parts checks, and a marked beam pair passes them as today.
  It refuses a grouping project, naming `lyx beam create <slug>`.
  It keeps refusing a task whose pair carries the beam mark, now naming `lyx batten cancel <slug> --remove`, since only an interrupted promote of an older build leaves one.
  The land refusal leaves the start guard and the pawl run's own Worktree-Create; `lyx beam land` and Beam-Await (D9, D10) apply it.
- Worktree-Create makes a beam project's pair from `boardengine.ParentBeam`'s worktree through `Topology.Add`, unmarked; Beam-Up marks it.
  The board's beam flag, written first by `lyx beam create`, holds a member started meanwhile at its Worktree-Create, as today.
- Dobby starts tasks only; a beam project's run is started by `lyx beam create` and resumed by adopt or a hand pass.
- A pawl run's owner verdict reads the beam project's own owner.
- `lyx batten cancel <beam>`, with or without `--remove`, refuses while `BeamLandRefusal` refuses, naming the members to land or cancel first.
  A cancel of a beam with no members ends the pawl run's processes and the beam's dobby and orch with them; `lyx beam create <slug>` starts the run again.
- Batten run records stay where they are: every run's batten record sits in the prime's records under its own slug's directory, also for a beam's members, so two machines driving runs of different projects write different paths of one records branch, which the records push's replay already merges, as two machines owning different tasks do today.

### D12 — The "beam can land" notice

A beam's dobby checks, whenever a run it drives reaches done after its teardown or leaves through a cancel with `--remove`, whether `BeamLandRefusal` now passes.
When it does, the dobby posts its own orch one notice naming `lyx beam land <slug>`; when another machine owns the parent, the notice says so and names `lyx adopt <parent>` (`lyx adopt root`) instead.
One notice per episode; a member opened again re-arms it.
The prime's dobby posts none.

### D13 — `lyx beam promote` goes

The verb, its `battencli` helpers (`EndRunForPromote`, `ClearRunForPromote`) and every way forward naming it are removed.
`fabricengine.ResetRecordsToParent` goes with it when nothing else calls it.
Rationale: a beam is made from its project, never from a running task.

## Constraints

- `PATTERN-owner-machine`: changed (see PATTERN changes); this machine's name only from `internal/machinename`, owners written only by `boardengine`'s owner setter, the start's owner read from the Bolt it just pulled.
- `PATTERN-config-layers`: only `hubreconcile.Ensure` reconciles after a binary change; it moves from `lyx orch start` to `lyx beam start`, and Beam-Up and adopt do not run it.
- `PATTERN-shed-recipe-registry`: Beam-Up, Beam-Await and Beam-Down go into the one `Constructor` map in `internal/shedrecipe`.
- `PATTERN-parent-lock`: only Finalize takes the parent lock; Parent-Align takes none.
- `PATTERN-pair-parent`: Worktree-Create of a beam project picks its parent through `ParentBeam`.
- `PATTERN-run-process-owner`: the dobby stays registered in `runproc` and is ended only through `runproc`, by its role (D4).
- `PATTERN-daemon-wakeups`: Beam-Await's wait.
- `PATTERN-refusal-way-forward`: every refusal above names its way forward, and a test reaches it.
- `PATTERN-tolerant-records`: a task's leftover `owner_machine`, a seed's `beam` recipe and a land request naming another run are read, never refused.
- `CLAUDE.md`: docs in the same commit — `beamcli`, `orchcli`, `adoptcli`, `boardengine`, `dobbyengine`, `battenshed`, `battencli`, `shortcutcli` `doc.go`, `docs/overview.md`, the orch stencils and `plugins/ly` skills.

## Auto-mode assumptions

- D5: grouping projects carry no owner (auto-pick).
- D1, D5: `root` names the prime; `board.json` keeps `prime_owner` (auto-pick).
- D4: `lyx s` takes no slug (auto-pick).
- D4: `lyx beam stop` takes no `--all` (auto-pick).
- D10: `lyx beam create` runs from the parent worktree only (auto-pick).
- D7: the orch watcher stops an orch whose beam another machine owns (auto-pick).
- D9: Beam-Await waits in-process (auto-pick).
- D10: a legacy beam migrates by re-running `lyx beam create <slug>` (auto-pick).

## Open risks

None.

## Acceptance criteria

- `lyx beam start` in the prime and in a beam starts a missing dobby and a missing orch, leaves live ones alone, and a second call starts nothing.
- A start with a recorded session resumes it; with a recorded session that does not come up it starts fresh from the last note and names the session it gave up; with neither it starts from the start stencil.
- `lyx beam start` refuses in a task pair naming its beam and path, and on a beam another machine owns naming `lyx adopt`.
- `lyx beam start --all` starts the prime and every local owned beam, and one failure leaves the others started.
- `lyx s` attaches after a successful start, and on a refusal exits non-zero without attaching.
- `lyx beam stop` stops orch, watcher and dobby and leaves a pawl run's driver and agents running; `lyx beam status` shows both parts.
- `lyx orch start`, `lyx orch cleanup`, `lyx prime`, `lyx beam promote` and `lyx adopt --prime` are gone, and no way forward, stencil or skill names them.
- A task's effective owner is its beam's owner, or root's; a task's leftover `owner_machine` is ignored and gone after the next board write; a recorded beam owner holds on a board with no root owner.
- A run in flight whose old task owner differs from its project's owner is owner-paused naming `lyx adopt <project>`.
- `lyx adopt G` moves G and its members' runs and starts G's dobby and orch, leaves a nested beam and G's parent alone, and does not resume G's own pawl run while another machine owns the parent.
- `lyx adopt root` with a nested beam owned elsewhere reports that beam's pawl run as owned elsewhere, not as an error.
- `lyx adopt <task>` and `lyx adopt <grouping project>` refuse naming the beam or `root` to adopt.
- An orch whose beam another machine owns is stopped by its watcher.
- `lyx beam create` records owner, beam flag and `recipe: pawl` and starts the batten run; the run makes the pair, Beam-Up marks it and starts dobby and orch, and the run waits at Beam-Await without quiet or driver-dead notices.
- While the pawl run is at Beam-Await, a member forks from the beam, verbs run inside the beam, and the project entry can be edited; once past it, a member released then is refused at its Worktree-Create.
- `lyx beam create --anchor <dir>` gives the pair that anchor, also across a resume of Worktree-Create; a create whose first pass a pause refused starts on a re-run after the lift.
- `lyx beam land` refuses with open members, with member pairs, on a beam or parent another machine owns, and on a legacy beam; otherwise Beam-Await ends, the beam's dobby stops, and the run goes through Parent-Align, the perch, Beam-Describe, Finalize and Beam-Down to Land and teardown.
- A land request with a member opened after it is dropped and posted to the beam's orch, and the run keeps waiting; a request left from an earlier pawl run is dropped.
- `lyx batten cancel <beam>` refuses while the beam has members.
- Pawl's Finalize resolves a conflict in the conflict session and lands an overlapping merge through the gate, with no reroute.
- A seed recording recipe `beam` resumes as pawl.
- A legacy beam with no pawl run gets one from `lyx beam create <slug>`.
- The beam's dobby posts one "can land" notice after its last member's teardown, naming `lyx adopt <parent>` when another machine owns the parent.
- Batten's start guard starts a beam project, refuses a grouping project naming `lyx beam create`, and refuses a task whose pair carries the beam mark.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run: only where its entry is not owned elsewhere; owners only on root and beam projects, set only by `boardengine`'s owner setter; any other entry reads its `hubgeom.ParentOf` owner; a dobby continue pass is no resume. — [background](pattern/PATTERN-owner-machine.md)

## Notes for the plan writer

- Pawl takes beam's Finalize flags in `landingdeps.go` (`LargeMergeLands`, no `ConflictReroutes`) and needs a review-follows seam now that it has a perch.
- The darn recipe's Code-Review rows, with `_lyx/discussion/board-entry.md` as answer key, are the model for pawl's perch.
- `PATTERN-owner-machine`'s background file needs its per-task owner and run-held paragraphs rewritten with the entry.
