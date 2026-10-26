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
Every `EnsureLive` caller starts only the beams it names: `lyx beam start` its beam, or each beam of `--all`; adopt its adopted projects (D6); Beam-Up its beam (D8); Beam-Await's failed accept its beam (D9); the anchor move the dobbies it ended.
None of them, nor batten's start of a run's parent dobby (`ensureParentDobby`), starts a dobby for a beam whose pawl run holds the landing hold (D9).

### D2 — The orch at start: resume, else note, else stencil

A live orch strand and a live watcher are left alone; a live strand with a dead watcher gets a watcher; a missing orch is started.
A started orch resumes the session the orch state records; with none recorded it starts fresh from the last completed handoff note; with no note, from the start stencil.
A recorded session whose resume does not come up live, or whose provider transcript is gone, falls back to the fresh start from the note, never to an error, and the envelope names the session it gave up.
A missing dobby is started on its own; the two parts are judged separately.
A fresh orch comes only from `lyx orch refresh` on a live orch, since `lyx orch stop` keeps the recorded session id and the next start resumes it.
Bound: the orch resumes the context the recorded id names; after a reed server rebirth in clear mode that is the pre-clear context, as `--continue` resumes it today.

### D3 — `lyx beam start` refusals

Its refusals come before any beam step.
The hub, task-pair and unknown-slug refusals come before the reconcile; the owner, anchor-move and landing-hold refusals come after the reconcile and the root claim, before any dobby, window or orch start.
It refuses:

- outside a hub, naming the hub;
- in a worktree that is no beam (a task pair), naming the beam it belongs to (its parent through `hubgeom.ParentOf`) and that beam's path, and `lyx beam start` there;
- a slug that names no local beam pair, naming `lyx beam create <slug>` from its parent worktree, or `lyx adopt <slug>` when its branches are on origin;
- a beam another machine owns, read from the Bolt the reconcile just pulled, naming the owner and `lyx adopt <slug>` (`lyx adopt root` for the prime);
- a beam with an anchor move standing, naming the move's way forward, as `EnsureLive` does today;
- a beam whose pawl run holds the landing hold (D9), naming `lyx batten status <slug>`; this replaces today's refusal of a beam whose entry its landing run holds.

### D4 — `lyx s`, `lyx beam stop`, `lyx beam status`, and what `lyx orch` keeps

- `shortcuts.yaml` gains `s: [[beam, start], [reed, attach]]` and drops `prime`.
  `lyx s` takes no slug: every word goes to `reed attach` as the shortcut routing does today, so it acts on the acting worktree's beam.
  A refusal exits non-zero, attaches nothing and leaves its envelope on the terminal; on success the attach covers the output.
- `lyx beam stop [slug]` stops the beam's orch strand and watcher through orch's stop function, then its dobby, each only when live; idempotent.
  The dobby is ended through `runproc` by its `RoleDobby` record alone, never by ending every process of the beam's slug, which would end the pawl run's driver and agents too.
  `runproc` has no role-scoped end (`End`, `EndSparing` and `EndProcess` only, `internal/runproc/end.go`), so it gains one that ends the live records of one role under an owner, with `End`'s signal sequence and bound.
  Bound: a stopped dobby does not drive the beam's runs, and passes already spawned finish their one pass; a hand `lyx batten run` there restarts the dobby, as it does today, unless the beam holds the landing hold (D1, D9).
- `lyx beam status` prints, per beam, the dobby section it prints today and the beam's orch: strand live or not, watcher live or not, recorded session id.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw`, `rules` and the hidden `watch` and `resume-context`.
  `lyx orch start` and `lyx orch cleanup` are removed; `lyx cleanup` already removes old handoffs.
  The `--adopt` launch stencil (`orch-template-adopt.md`) goes with `--adopt`.
- Every way forward, notice, stencil and skill naming `lyx orch start` or `lyx prime` names `lyx beam start`, under fb5c10c3's rule that messages name the full command, never a shortcut.

### D5 — Ownership sits on projects

- The root project's owner is the board's `prime_owner` key, kept under that name in `board.json`; a beam project's owner is its `owner_machine`.
- A task and a grouping project record no owner.
  Their effective owner is the owner of `hubgeom.ParentOf`'s answer: the nearest beam they belong to, or root.
  This narrows the entry's "each project is owned by one machine" to beam projects and root: a grouping project has no worktree and no dobby to drive it, so its tasks are driven by the beam or prime they fork from.
- `Owners.Effective`: root reads `prime_owner`; a beam project reads its recorded owner, else its parent's; any other entry reads its parent's owner.
  On a board with no root owner, a recorded beam owner holds for the beam and every entry whose `ParentOf` answer is that beam, and every other entry reads as owned by no other machine.
  The run-held rule (a run-held entry with no owner reads as no other machine) goes, from `Effective` and from every reader that applies it on its own, such as tender's owner-pause selection and adopt's test of a started entry, which reads the run status.
- A task's or grouping project's `owner_machine` is dropped on load, the way `migrate.go` drops `tier` and `type`, so it leaves `board.json` at the first write after the upgrade and is never read.
- `boardengine`'s owner setter writes only the root owner and beam project owners; its beam write sets the beam flag and `recipe: pawl` in the same commit.
  Batten's start claim writes the run status alone and refuses a task whose effective owner is another machine; its resume claim writes no owner either, so `resumeOwnerClaim`'s owner write goes.
- The list and get views carry the effective owner as today; the README's Running Owner column shows the effective owner.
- Upgrade: a run in flight whose recorded task owner differs from its project's owner reads its project's owner after the upgrade, so the machine driving it owner-pauses it at its next pass, naming `lyx adopt <project>`.
  Bound: every machine of the hub runs the same build, deployed through `update-plugins.sh`; until the other machine is updated it still reads and writes task owners, which the new build drops again on its next load.
  An older build refuses a project carrying a recipe (`checkProjectFields`, `internal/boardengine/task.go`, run by `normalizeTask` on upsert and on replay in `replay.go`), so once a beam project carries `recipe: pawl` that build's upsert, patch or merge of the entry fails, and so does the replay of its own local commits that touch it, its other entries unaffected, until the machine is updated through `update-plugins.sh`.
  Its `SetRunStatus` and owner-setter writes to the entry skip `normalizeTask` and still succeed locally, and reach origin whenever their push needs no replay, so an older machine can still overwrite the beam project's run status and owner until it is updated through `update-plugins.sh`.
Rationale: a task follows its project, whose owner's dobby starts and drives it.

### D6 — `lyx adopt <project>...` and `lyx adopt root`

- The arguments are beam project slugs and `root`; `--all` names root and every open beam project; `--keep-local` and `--drop-local` stay.
  `--prime` is removed.
  This reads the entry's "There is no adopt of a beam": a beam has no adopt of its own; `lyx adopt` names the beam's project, and the pair, dobby and orch move with it.
- Refused before anything runs: a task, naming `lyx adopt <its beam or root>`; a grouping project, naming `lyx adopt <its beam or root>`; a note and an unknown slug as today; a project that is no beam, naming `lyx beam create <slug>`.
- An adopt moves the named project alone: its owner, its pair (added when absent and its branches are on origin) and the runs whose parent is its worktree.
  A nested beam keeps its own owner, so `lyx adopt root` moves root's tasks, its grouping projects' tasks and their runs, and never a beam below it.
- The refusals that a parent or a member reads owned by another machine go: no task has an owner, and a beam may be owned apart from its parent.
- After the owner write, adopt resumes each run in flight whose parent is an adopted worktree, as a detached first pass.
  It also resumes a beam's pawl run whenever, after the owner write, this machine owns both the beam and its parent, whichever of the two the adopt named, so adopting root then G and adopting G then root both leave G's pawl run driven here.
  A run whose entry reads owned by another machine, such as a nested beam's pawl run under an adopted root, is not resumed and is reported as owned elsewhere, never as an error.
  Bound: until the same machine owns both a beam and its parent, the beam's pawl run gets no batten pass on either machine, since the old parent owner's dobby exits on its owner read and the adopter resumes no run it reads owned elsewhere, and `lyx batten status <beam>` shows it at its last row, undriven; its landing stays refused by D10's parent-owner refusal, and `lyx adopt <beam>` on the parent's owner, or `lyx adopt <parent>` (`lyx adopt root`) on the beam's owner, resumes it.
- Adopt then runs each adopted project's beam steps (D1), since adopt's own step 1 already reconciled origin.
  It skips them for a beam whose pawl run holds the landing hold (D9), still writes its owner, and reports that beam naming `lyx batten status <slug>`; when this machine does not also own the beam's parent, the report names `lyx adopt <parent>` (`lyx adopt root`), which resumes the landing here by the rule above.
- The origin reconcile, `--keep-local`/`--drop-local`, the anchor-move refusal, the owner pause lift and the machine check (step 6) are unchanged.

### D7 — A beam's two parts leave a machine that no longer owns it

A beam's dobby already exits when it reads its beam owned by another machine.
The orch's watcher in a beam or the prime reads the beam's owner from the local Bolt on its tick, and when another machine owns it stops the orch strand and exits, logging the owner.
A failed owner read reads as not owned elsewhere and is logged once per episode, as the dobby's own owner read does, so the watcher stops the orch only on a read naming another machine.
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
| Finalize | `landingshed.Finalize` under the parent lock, after `BeamLandRefusal` passes again (D9); `ConflictReroutes` false, `LargeMergeLands` true, no `on_reroute` |
| Beam-Down | stops the beam's orch and watcher and, when still live, its dobby, as `lyx beam stop` does; a failure warns and is Done, since Worktree-Teardown ends what is left |

`profiles.yaml`'s template gains a `pawl` block per profile, carrying the perch's settings.

### D9 — Beam-Await and the beam's state after it

- Beam-Await waits in its own driver process, waking on the land request file and on a timer at most once a second with backoff to a minute while idle (`PATTERN-daemon-wakeups`); it spawns no agent.
- A land request names the pawl run it is for, by the run's seed id; Beam-Await accepts only a request naming its own run and removes any other.
- On entry Beam-Await writes the entry's run status naming Beam-Await, and again at each wake until a write succeeds.
- On an accepted request it first writes the entry's run status naming Parent-Align, so the landing hold stands and no start of the beam's dobby passes (D1, D3); then writes a dobby-stop marker under the beam worktree's `.lyx/beam/` and stops the beam's dobby, so it starts no new member; then re-runs `lyx beam land`'s checks: `BeamLandRefusal`'s members and member pairs, the beam's owner and its parent's owner (D10).
  The run-at-Beam-Await check is not re-run, since the row running the accept is that check.
  A member's Worktree-Create reads the hold again once its pair is made and, when the hold stands, removes the pair and is refused, so either that read or the re-check sees the other; a member a still-live dobby starts between the hold write and the dobby stop is refused by that read.
  When the checks pass it writes the project entry snapshot `_lyx/discussion/board-entry.md`, the perch's answer key, removes the request, then the marker, and is Done.
  When they fail it writes the run status naming Beam-Await again, restarts the beam's dobby whenever the marker stands and then removes the marker, removes the request, posts the failure to the beam's own orch and keeps waiting.
  A failed dobby restart leaves the marker standing and still removes the request, and the failure posted to the beam's orch names both the re-check's blockers and `lyx beam start <slug>`, which passes since the status write lifted the hold; the next accepted request restarts the dobby on its own failed re-check, since the marker still stands.
  A failed status write keeps the request, and the next wake retries the accept; the marker carries over, so an accept retried, or re-entered after a crash between the dobby stop and the request's removal, still restarts the dobby on a failed re-check.
- Finalize re-runs `BeamLandRefusal` under the parent lock and, when it refuses, halts with ways forward it words itself from the refusal's blockers, never `BeamLandRefusal`'s land-it wording, which the standing hold refuses.
  Per blocker it names: for a member with a pair or a run in flight, and for a pair forked from the beam, `lyx batten cancel <slug> --remove`, run in the beam's parent worktree, or on the machine holding the pair when the run is in flight elsewhere; for a member with no run status, moving its `project` off the beam with `lyx board upsert`, which the run lock would refuse for a run-held member; then `lyx loom resume` in the beam worktree.
  It is a backstop for an open member the re-check missed, such as one a stale Bolt on another machine added.
  Bound: a member that lands into the beam before Finalize is not caught there; the Worktree-Create read above keeps any member from forking once the hold stands.
- A Beam-Await entered with no request standing and the entry's run status naming Parent-Align, as a crash between the request's removal and Done leaves it, posts the beam's orch one notice that its landing was interrupted, naming `lyx beam start <slug>` and `lyx beam land <slug>`; its entry write above lifts the hold first, so both steps run, and it removes a dobby-stop marker left standing.
- When it reads its beam owned by another machine it removes any request and ends its driver with a yield, deciding nothing; the batten pass's owner check owner-pauses the run.
  Bound: until the owner pause stands, a batten pass may re-spawn the driver once per probe pass, each yielding at once.
- Batten's InnerRun sends no quiet notice for a child at Beam-Await, as it sends none while a parent lock wait record stands; a restarted batten re-spawns the driver, which re-enters the row.
  InnerRun's existing driver-dead notice applies at Beam-Await as at any row (`internal/battenshed/notice.go`): a probe that finds the running child's driver dead sends it, once per state episode of the child.
  Once `respawnCap` respawns have passed with no status-file write between them (`internal/battenshed/innerrun.go`), InnerRun spawns no further driver.
  The cap changes only the notice's text, and its episode key is the same at and below the cap, so the text naming `lyx loom start` goes out only when the cap is reached before any driver-dead notice of the episode; otherwise the episode's one notice is the plain "driver strand is dead".
  A re-spawned driver that writes its status file on re-entering the row resets the respawn count, so the respawns go on.
- The landing hold, stated here once: the pawl run holds none from Worktree-Create through Beam-Await, a halt at any of those rows included, and holds one from Parent-Align on.
  Every reader judges it from the producer half of the beam project entry's run status (`<state> · <producer>`).
  While the run holds none, its run status reads as no hold to every reader that asks whether the entry is held from change: membership (`checkProjectField`) accepts a task joining the project, the entry stays in the board's Project layer, the run lock leaves its fields editable, save that a beam project entry with a pawl run in flight refuses at every row a change of `project`, `kind`, `recipe` and `beam`, its removal and its merge, naming `lyx batten cancel <slug>`, while title, brief, body, order and labels stay editable; members fork from the beam (`beamForkPrecondition`), every verb runs inside the beam (`ResolveBattenPrime`'s landing refusal), the beam starts (D3), and `lyx loom peers` lists it as no in-flight peer.
  Once it holds one, each of these refuses or locks as today's run hold does, so a member released then is refused at its Worktree-Create.
  Every reader that asks whether a run is in flight sees the pawl run in flight at every row, hold or none: adopt's test of a started entry, tender's owner-pause selection, batten's owner check and the parent dobby's driven runs.
  Bound: the driver's own transition writes are warn-only; the writes that cross the hold are Beam-Await's, each retried at the next wake until it succeeds, so a status left on the wrong side of the hold stands at most until that wake.
- A pawl run takes no slot under its parent dobby's `max_runs` while it holds no landing hold, and takes one from Parent-Align on; the parent dobby still schedules its passes, owed full passes and probes, as for every run it drives.

### D10 — `lyx beam create` and `lyx beam land`

- `lyx beam create <slug> [--anchor <dir>]` runs from the beam's parent worktree, the prime or the parent beam, and refuses elsewhere naming that worktree's path.
  It takes an open project entry, refuses a note, a task and a done project as today, refuses a beam another machine owns naming `lyx adopt <slug>`, and refuses when the parent reads owned by another machine, naming `lyx adopt <parent>`.
  It claims the root for this machine when the board records no root owner, records through the owner setter this machine as the beam's owner with the beam flag and `recipe: pawl`, then starts the batten run as a detached first pass, driven on by the parent's dobby, and names `lyx batten status <slug>`.
  `--anchor` rides into batten's own run record at that first pass, and Worktree-Create passes it to `Topology.Add`, so a resume keeps it.
  Re-run, it skips what is done: a pawl run in flight is reported live; a flagged beam with no run status, left by a first pass a covering pause refused or that died, gets its first pass again; a legacy beam with a marked pair and no pawl run gets its pawl run, Worktree-Create finding the pair complete and Beam-Up finding it marked.
  Bound: a first pass refused for a covering pause leaves no run, the envelope names the pause, and the same `lyx beam create <slug> [--anchor <dir>]` after the lift starts it; a flagged beam with no run status left by a first pass that died likewise takes the same `--anchor` again, since its anchor reached no run record; dobby never starts a beam project.
- `lyx beam land <slug>` runs anywhere in the hub and writes a land request naming the pawl run in the beam worktree's `.lyx/beam/`, which Beam-Await consumes.
  It refuses: while any open entry's `hubgeom.ParentOf` is the beam, or any local pair's origin record names the beam as its parent worktree, naming each with `BeamLandRefusal`'s ways forward; a beam this machine does not own, naming `lyx adopt <slug>`; a beam whose parent another machine owns, naming `lyx adopt <parent>` (`lyx adopt root` for the prime); a beam whose pawl run is not running at Beam-Await, naming `lyx batten status <slug>`; a legacy beam with no pawl run, naming `lyx beam create <slug>`.
  `BeamLandRefusal` itself loses its orch and watcher blocker (`beamLandRefusal`'s `orchLive` and `watcherLive` cases, `internal/battencli/beamland.go`), since the beam's orch runs until Beam-Down, so every caller (`lyx beam land`, Beam-Await's re-check, Finalize, `lyx batten cancel <beam>` and the dobby's can-land check) judges members and member pairs alone.
- Today's separate landing run of `lyx beam land` goes.
- The board accepts `recipe: pawl` on a project only, and refuses it on a task; a beam project with no recipe reads as pawl.

### D11 — Batten for a beam project

- Batten's start guard lets a beam project start or resume: a fresh start gets a task's dirty-source and left-parts checks, and a marked beam pair passes them as today.
  It refuses a grouping project, naming `lyx beam create <slug>`.
  It keeps refusing a task whose pair carries the beam mark, now naming `lyx batten cancel <slug> --remove`, since only an interrupted promote of an older build leaves one.
  The land refusal leaves the start guard and the pawl run's own Worktree-Create; `lyx beam land`, Beam-Await and Finalize (D9, D10) apply it.
- Worktree-Create makes a beam project's pair from `boardengine.ParentBeam`'s worktree through `Topology.Add`, unmarked; Beam-Up marks it.
  The board's beam flag, written first by `lyx beam create`, holds a member started meanwhile at its Worktree-Create, as today; the pawl run's own status there is no landing hold (D9).
- Dobby starts tasks only; a beam project's run is started by `lyx beam create` and resumed by adopt or a hand pass.
- A pawl run's owner verdict reads the beam project's own owner.
- `lyx batten cancel <beam>`, with or without `--remove`, first runs the hub-scope origin reconcile and refuses while `BeamLandRefusal` refuses over the Bolt it pulled, naming the members to land or cancel first; when origin is unreachable it refuses with the reconcile's way forward.
  A cancel of a beam with no members ends the pawl run's processes and the beam's dobby and orch with them; `lyx beam create <slug>` starts the run again.
- Batten run records stay where they are: every run's batten record sits in the prime's records under its own slug's directory, a beam's members' included.

### D12 — The "beam can land" notice

A beam's dobby checks whether `BeamLandRefusal` now passes on every board change it already ticks on, and whenever a run it drives reaches done after its teardown or leaves through a cancel with `--remove`, so a member moved off the project or marked done by hand also triggers it.
When it does, the dobby posts its own orch one notice naming `lyx beam land <slug>`; when another machine owns the parent, the notice says so and names `lyx adopt <parent>` (`lyx adopt root`) instead.
One notice per episode; a member opened again re-arms it.
The dobby's state record keeps the sent-notice episode, so a restarted dobby sends no second notice.
The prime's dobby posts none.

### D13 — `lyx beam promote` goes

The verb, its `battencli` helpers (`EndRunForPromote`, `ClearRunForPromote`) and every way forward naming it are removed.
`fabricengine.ResetRecordsToParent` goes with it when nothing else calls it.
Rationale: a beam is made from its project, never from a running task.

### Added after Discussion (parent, 2026-10-26): runproc's End family ends a run's processes

Decision: PATTERN-run-process-owner changes so ending a dobby through the new runproc.EndRole is within the invariant; the background file's opening sentence changes with it.

Rationale: EndSparing already exists beside End, and the plan's EndRole is the same family; the invariant is that runproc alone ends a run's processes, not one function.

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

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - accept-dobby-restart-failure-unbounded (class design, severity MEDIUM, open in rounds 3)
  - beam-await-driver-respawn-cap-silent (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - battencli-resume-claim-tests-unnamed (class scope, severity MEDIUM, open in rounds 2)
  - localownerverdict-no-adopt-subject (class scope, severity MEDIUM, open in rounds 2)
  - orch-role-stencil-adopt-slug (class scope, severity MEDIUM, open in rounds 2)
  - run-process-owner-background-endrole-drift (class decision, severity MEDIUM, open in rounds 1, 2)
  - testrunownerverdict-on-no-card (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `lyx beam start` in the prime and in a beam starts a missing dobby and a missing orch, leaves live ones alone, and a second call starts nothing.
- A start with a recorded session resumes it; with a recorded session that does not come up, or whose provider transcript is gone, it starts fresh from the last note and names the session it gave up; with neither it starts from the start stencil.
- `lyx beam start` refuses in a task pair naming its beam and path, and on a beam another machine owns naming `lyx adopt`.
- `lyx beam start --all` starts the prime and every local owned beam, and one failure leaves the others started.
- `lyx s` attaches after a successful start, and on a refusal exits non-zero without attaching.
- `lyx beam stop` stops orch, watcher and dobby and leaves a pawl run's driver and agents running; `lyx beam status` shows both parts.
- `lyx orch start`, `lyx orch cleanup`, `lyx prime`, `lyx beam promote` and `lyx adopt --prime` are gone, and no way forward, stencil or skill names them.
- A task's effective owner is its beam's owner, or root's; a task's leftover `owner_machine` is ignored and gone after the next board write; a recorded beam owner holds on a board with no root owner.
- A run in flight whose old task owner differs from its project's owner is owner-paused naming `lyx adopt <project>`.
- `lyx adopt G` moves G and its members' runs and starts G's dobby and orch, leaves a nested beam and G's parent alone, and resumes G's own pawl run only when this machine also owns G's parent.
- Adopting root then G, and G then root, each leave G's pawl run resumed on the adopting machine.
- Adopting a beam whose pawl run holds the landing hold writes its owner, starts no dobby or orch for it and names `lyx batten status <slug>`, and `lyx adopt <parent>` when another machine owns the parent.
- `lyx adopt root` with a nested beam owned elsewhere reports that beam's pawl run as owned elsewhere, not as an error.
- `lyx adopt <task>` and `lyx adopt <grouping project>` refuse naming the beam or `root` to adopt.
- An orch whose beam another machine owns is stopped by its watcher.
- `lyx beam create` records owner, beam flag and `recipe: pawl` and starts the batten run; the run makes the pair, Beam-Up marks it and starts dobby and orch, and the run waits at Beam-Await without quiet notices.
- While the pawl run is at Beam-Await, a member forks from the beam, verbs run inside the beam, the project entry's title, brief, body, order and labels can be edited, and a task upserted with `project` set to the beam is accepted; once the run holds the landing hold, a member released then is refused at its Worktree-Create.
- A parent dobby with `max_runs` pawl runs at Beam-Await still starts a ready task.
- A member whose Worktree-Create makes its pair after Beam-Await's hold write is refused and its pair removed; an open member Finalize finds halts the pawl run naming its cancel when it has a pair or a run in flight, else its move off the beam, then `lyx loom resume`, and never its landing.
- A Beam-Await re-entered after a crash between the request's removal and Done lifts the hold and posts one interrupted-landing notice; a Beam-Await re-entered with its status naming Beam-Await and no request standing posts none.
- An accept re-entered after a crash between the dobby stop and the request's removal, whose re-check fails, leaves the beam's dobby live.
- A failed re-check whose dobby restart fails leaves the dobby-stop marker standing and posts the beam's orch a failure naming the re-check's blockers and `lyx beam start <slug>`.
- A Beam-Await driver found dead sends one driver-dead notice in its state episode, also when it dies after re-entering the row and writing its status.
- While the pawl run is at Beam-Await, an upsert moving the beam project's `project` to another parent is refused naming `lyx batten cancel <slug>`.
- `lyx beam create` refuses a beam another machine owns naming `lyx adopt <slug>`, and a beam whose parent another machine owns naming `lyx adopt <parent>`.
- `lyx beam create --anchor <dir>` gives the pair that anchor, also across a resume of Worktree-Create; a create whose first pass a pause refused starts on a re-run after the lift.
- `lyx beam land` refuses with open members, with member pairs, on a beam or parent another machine owns, and on a legacy beam; otherwise Beam-Await ends, the beam's dobby stops, and the run goes through Parent-Align, the perch, Beam-Describe, Finalize and Beam-Down to Land and teardown.
- A land request with a member opened after it is dropped and posted to the beam's orch, and the run keeps waiting; a request left from an earlier pawl run is dropped.
- `lyx batten cancel <beam>` refuses while the beam has members.
- Pawl's Finalize resolves a conflict in the conflict session and lands an overlapping merge through the gate, with no reroute.
- A seed recording recipe `beam` resumes as pawl.
- A legacy beam with no pawl run gets one from `lyx beam create <slug>`.
- The beam's dobby posts one "can land" notice after its last member's teardown, naming `lyx adopt <parent>` when another machine owns the parent; after that notice, opening a member and landing it again sends a second notice.
- Batten's start guard starts a beam project, refuses a grouping project naming `lyx beam create`, and refuses a task whose pair carries the beam mark.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run: only where its entry is not owned elsewhere; owners only on root and beam projects, set only by `boardengine`'s owner setter; any other entry reads its `hubgeom.ParentOf` owner; a dobby continue pass is no resume. — [background](pattern/PATTERN-owner-machine.md)
- `PATTERN-run-process-owner` — changes: A run's processes: registered in `internal/runproc`, ended only via `runproc`'s `End`, `EndSparing` or `EndRole`. — added after Discussion (parent, 2026-10-26)

## Notes for the plan writer

- Pawl takes beam's Finalize flags in `landingdeps.go` (`LargeMergeLands`, no `ConflictReroutes`) and needs a review-follows seam now that it has a perch.
- The darn recipe's Code-Review rows, with `_lyx/discussion/board-entry.md` as answer key, are the model for pawl's perch.
- `PATTERN-owner-machine`'s background file needs its per-task owner and run-held paragraphs rewritten with the entry.
