# Decision record — umbrella-groundwork

## Goal

Give the hub beams: long-lived pairs that group board work, spawn their tasks from their own branch and take them back, each with an optional orch of its own.
The groundwork comes first: board project entries, a fabric pair created from a named parent pair, and batten and orch running from any worktree of the hub.
The word is beam throughout, never umbrella, in every name, text and doc this task touches.

## Scope

In:

- Board: a third entry kind, `project`, with a `beam` flag and an ordered member list; a `project` field on tasks and notes; README grouping by project; the project chain as the source of a new pair's parent and of an unspawned entry's owner.
- Board: one slug rule from fabric, applied at the writes that make a task or project slug (issue #724).
- Fabric: `AddOptions.Parent`, a pair created from a named parent pair's branch; `lyx fabric add --adopt-local` over an existing local branch; a beam mark in the origin record.
- Fabric: a VS Code workspace file per beam beside the hub's, each listing one level.
- Batten, `lyx adopt` and `lyx cleanup` run from the prime and from any beam, with batten's state kept under the prime; Worktree-Create forks from the task's beam.
- `lyx batten cleanup`'s skip of a run that is not done names `lyx batten cancel <slug>` (D16).
- Notices and parent names go to the nearest live orch up the beam chain, bar the ready notice, which goes only to the task's own orch (D13).
- Orch: `lyx orch` runs from the prime and from any beam, one orch per worktree, named `<shortname>:<slug>:orch` in a beam.
- A new module `lyx beam` with `create`, `promote` and `land`, and a landing recipe `beam`.

Out:

- `lyx beam start` and dobby, the beam daemon, which are `umbrella-daemon`'s, as is the later removal of the ready notice D13 routes.
- An anchor per beam (`umbrella-anchor`), Weaver's batch pairs (`weaver`, `parallel-batch-parts`) and the `stitch` pieces (`loom-beam-darns`); they reuse `AddOptions.Parent` and the beam module later.
  `loom-beam-darns` turns a task into a beam while its run continues, which D10's promote, ending the run, does not do; that conversion is that entry's own design.
- The ready-to-land handover through Bolt between machines: no landing has it today, the prime's included, and Finalize's owner checks already keep a landing into a beam on the beam's owner machine.
- Reed grouping of a beam's sessions under it.
- `lyx orch` in a task pair that is not a beam: a pair's tasks come from a beam by D4, so an orch anywhere else has no tasks of its own.
- `lyx shed seed` stays prime-only.
- The blocked-run cleanup half of issue #724, which `lyx batten cancel` already covers.

## Decisions

### D1. A project is a third board kind

Decision: `kind: project` joins task and note.
A project carries a title, brief, body, labels (one or more type labels, as a task), an optional `project` of its own naming its parent project, an `order` list and a `beam` flag.
It carries no `depends_on`, `recipe`, `profile`, `run_settings` or `priority`, and a write giving it one is refused, naming the field to drop.
No task may depend on a project, as no task may depend on a note today; the refusal names the project's task members as what to depend on instead.
The refusal judges only a write that sets or changes `depends_on`, as D2's rules do for `project` and `order`, so a done task whose unchanged `depends_on` names a promoted project (D10) refuses nothing.
Its handle letter is `P`; `LoadConfig` refuses a type label whose letter is `P`, as it refuses `T`, naming the label and renaming it in `board.yaml` as the way forward.
Only a task is claimable, bar the landing run of a project whose `beam` is true (D11).
The board's layering gives a project no task layer: an open project lists in `lyx board list` and `get` under a layer of its own, `Project`, never Ready or a dependency letter, bar a project its landing run holds, which lists as `Running` as a run-held task does, and no project counts as a dependency's depth.
A project carries `owner_machine` as a task does: the owner setter, `Owners.Effective` and the list view's owner treat a project like a task.
Rationale: the operator decided a project entry on the board; promote (D10) needs a kind change, so a flag on a task cannot carry it.

### D2. Membership: `project` on an entry, plan order on the project

Decision: a task and a note may carry `project`, the slug of a project entry; a project's `order` lists member slugs in plan order.
A write is refused when, for an entry whose `project` or `order` it sets or changes, it leaves:

- a `project` naming an absent entry, a non-project entry, the entry itself, a done project, a project whose status is a run status, or a project chain with a cycle;
- an `order` naming an entry whose `project` is not this project, or naming one entry twice.

Each refusal names the entry, the field and the slug at fault, with the way forward of correcting that slug or dropping it from the field.
An entry whose `project` the write leaves as it was is never refused by these rules, so marking a project done, by hand or at its landing, refuses nothing.
A change of `project` on an entry whose pair exists on this machine is refused, since the pair's origin record keeps naming the old parent, naming `lyx batten cancel <slug> --remove` as the step before the change.
A member missing from `order` renders after the ordered ones, by handle.
Promote keeps a note's `project`.
Rename rewrites every `project` and `order` naming the old slug.
Rename refuses a project whose `beam` is true, and any entry whose pair exists on this machine, since a pair's branch is derived from its slug; the refusal says a beam's slug stays fixed while it lives, naming `lyx board upsert` of its title as the way forward, and names `lyx batten cancel <slug> --remove` for any other entry with a pair.
Prune drops pruned slugs from every `order`, and keeps a done project while an entry the same prune does not remove names it.
The run lock lets prune's `order` rewrite through on a held project, as it lets prune's `depends_on` rewrite through today.
A merge carries the removed entries' memberships to the upserted entry and refuses to remove a project another entry still names, naming those entries as the ones to move first.
`project`, `order` and `beam` are otherwise scope, so the run lock refuses a change to them on a run-held entry.
Rationale: the operator decided both the per-entry field and the project's ordered list; notes are members because the `workspaces` project holds notes such as `umbrella-daemon` and `umbrella-anchor`.

### D3. The `beam` flag is written only by the beam verbs

Decision: an upsert, batch or merge payload may carry `beam` only as the entry's stored value, dropped as a no-op; any other value is refused naming `lyx beam create` or `lyx beam promote`.
`boardengine`'s owner setter gains the beam flag: `lyx beam create` and `lyx beam promote` record this machine as owner and set `beam: true` in its one Bolt commit, and are its only callers that pass the flag.
For `lyx beam promote` alone, the same commit also changes the entry's kind to project and clears its `depends_on`, `recipe`, `profile` and `run_settings` (D10).
Nothing sets it back: a landed beam is a done project.
Bound: a hand edit of `board.json` can still set it; batten then finds no beam pair and stops at Worktree-Create with D7's precondition, which, the beam's branch being absent from origin, names `lyx beam create <beam>`, and D9 admits that create on the machine the project's effective owner names, and elsewhere refuses it with its refusal of a beam another machine owns.
Rationale: a `beam: true` set without the verb would send every member to a pair that may never be made.

### D4. One parent resolver: origin record, else the project chain

Decision: `ParentBeam` in `boardengine` walks an entry's `project` links upward, skipping done projects, and returns the nearest project whose `beam` is true, or the empty slug for the prime.
One `ParentOf` built in `internal/hubgeom` is the only one every owner reader is handed, batten's, adopt's, loom's owner check, fabric remove's, pair teardown's and `boardcli`'s alike: it reads the slug's origin record and, only when no record is found, `ParentBeam` over the local board.
Origin found with an empty parent reads as the prime and never falls through to the board.
A found record's parent slug is its `ParentWorktree`, read as the prime when `ParentBranch` is the prime's branch or `ParentWorktree` names the prime's worktree, and never derived by matching `ParentBranch` against the pair worktrees present on this machine;
so a member whose beam pair is absent here still reads the beam as its parent, and its owner as the beam's owner.
Batten's Worktree-Create (D7) and `lyx beam create` (D9) use `ParentBeam` to pick a new pair's parent.
`lyx board list` and `get` carry each entry's `ParentOf` answer as `parent_beam`, empty for the prime, so on the machine holding an entry's pair its `parent_beam` is the parent the pair lands into, also for a member spawned under a grouping project that later became a beam.
Bound: on a machine without that pair, `parent_beam` falls back to `ParentBeam`; the entry's run is owned by the pair's machine, whose orch reads it right, and an orch elsewhere starts none of it, since batten refuses a start of an entry another machine owns.
A grouping project, one whose `beam` is false, inside a beam passes its members on to that beam.
Rationale: the operator decided that a task of a beam forks from and lands into the beam, a task whose project has no worktree goes to the prime, and every beam and task has its own owner with an unrecorded owner read from the parent; one resolver keeps the start, claim, landing and teardown verdicts alike.

### D5. Fabric creates a pair from a named parent pair

Decision: `AddOptions.Parent` names a pair slug of the hub; empty keeps today's fork from the acting worktree's HEAD.
Every caller that picks a parent from the board passes the prime's location as the acting location, so an empty `ParentBeam` forks from the prime wherever the caller runs.
With a parent named, `Add` forks the warp branch from the parent worktree's HEAD and the weft branch from the parent's weft branch, and records the parent's branch as `ParentBranch` and its worktree name as `ParentWorktree`.
It refuses, before any mutation, a parent that is not a pair of this hub, a parent worktree on a detached HEAD, and one whose checked-out branch is not its own pair branch, naming `lyx fabric checkout` as the way forward.
With `Parent` named, Add refuses no tracked dirt at all, neither the named parent's nor the acting worktree's, since the pair forks from the parent's last commit and never from the acting worktree, and it logs the parent's dirty paths at Info;
with `Parent` empty, Add keeps today's refusal of the acting worktree's tracked dirt.
`lyx fabric add <slug> --parent <slug>` exposes it.
Merge-back, the parent lock and teardown need no change: they already follow the origin record.
Bound: uncommitted edits in a beam never reach its members, and starting a member never waits on them.
Landing a member does wait on them: Finalize keeps its Stuck on a dirty parent pair, so a member whose beam holds tracked dirt waits at Finalize until the operator commits or stashes in the beam, the way forward that Stuck already names, and resumes through it.
Rationale: forking from a parent pair's branch lets any process create a beam's task from anywhere, and the operator works in the beam, so its dirt is routine, not a mistake.

### D6. `lyx fabric add --adopt-local` adopts an existing local branch

Decision: `AddOptions.AdoptLocal`, set only by `lyx fabric add`'s `--adopt-local` flag, lets `Add` make a pair over an unmarked local warp branch that already exists, instead of refusing `ErrBranchExists`.
Batten, adopt and every other caller leave it unset and keep the refusal, and the orch role stencil never passes the flag.
A branch checked out in another worktree is refused, naming that worktree and `lyx fabric checkout` to move it off; on the weft side the same refusal names the weft branch's worktree.
An adopted branch records its parent from `--parent` or the acting worktree as in D5, except the warp's default branch, which records an empty `ParentBranch` and `ParentWorktree`: a root pair, which batten never lands, since the Land row already stops on an origin record without a parent branch.
`ErrBranchExists`'s message names `--adopt-local`.
Bound: only a command that names the flag adopts a local branch, so an unattended run never adopts a stale branch left under a reused slug.
Rationale: there is no lyx way to make a pair over a branch already in the clone, and the default branch has no parent to record.

### D7. Batten's Worktree-Create forks from the task's beam

Decision: Worktree-Create resolves the task's parent with `ParentBeam` and calls `Add` with that slug as `AddOptions.Parent` and the prime's location as the acting location.
The beam's pair absent on this machine stops the row with a `PreconditionError` naming the beam and its way forward, picked by whether the beam's branch exists on origin: `lyx beam create <beam>` when it does not, which also covers a create stopped before its pair (D9), and `lyx adopt <beam>` when it does, whose pair then carries the origin record, beam mark included, from the beam's weft branch.
A beam whose board entry is run-held by its own landing stops the row with a `PreconditionError` naming the landing.
Batten's start guard keeps its tracked-dirt refusal for a pair forked from the prime, and checks none for one forked from a beam (D5) or for a beam's landing (D11).
Rationale: the operator decided that a project with a worktree is where its tasks spawn from and land into; the origin record then carries the rest.

### D8. Batten, adopt, cleanup and orch from the prime and any beam

Decision, batten, adopt and cleanup: `RefuseUnlessPrime` gives way, for every batten verb, `lyx adopt`, `lyx cleanup` and every `lyx beam` verb, to a guard that admits the prime and a beam, a pair whose origin record carries the beam mark (D12), and resolves the prime's location from it; each verb then acts on the prime's location as it does today.
The guard refuses every other worktree, a task pair that is not a beam included, naming as where to run the verb the nearest worktree up the pair's origin chain that exists and is a beam, else the prime.
`lyx adopt` admits a project whose `beam` is true as it admits a started task, taking over its owner and adding its pair when its branches are on origin.
What the wider guard lets through is a session in a beam's worktree, the operator's or the beam's orch, acting on any run of the hub as from the prime.
The beam test bounds it: an agent of a run works in that run's own pair, so a member's agents stay refused;
the only run whose agents work in a beam is the beam's own landing (D11), and the guard refuses in a beam whose board entry is run-held by its landing, naming the landing and the beam's parent worktree as where to run the verb, so the landing's merge fixer and conflict session reach no batten, adopt, cleanup or beam verb.
Every batten path, the status file, locks, scratch and the prime lock, stays under the prime's anchor.
A verb that creates or destroys a pair refuses when the acting worktree is that pair itself, naming the parent or the prime as where to run it.
`lyx batten run --window` opens its window in the reed session of the run's parent worktree, the prime's or a beam's, with the prime as its working directory, wherever the verb runs; a beam's session then holds only its members' windows, and D11's refusal keeps a beam from landing while a member pair exists.
Decision, orch: every `lyx orch` verb runs in the prime and in a beam, a pair whose origin record carries the beam mark (D12), and refuses elsewhere naming `lyx beam create` or `lyx beam promote`.
A beam's orch keeps its files under the beam's own `.lyx/orch/` and `_lyx/orch/`, runs in the beam pair's reed session, and is the strand `<shortname>:<slug>:orch`, the name `hubgeom.ResolveParent` already gives a pair forked from that worktree; its watcher is its own.
`lyx orch start` in a beam another machine owns refuses, naming the owner and `lyx adopt <slug>`.
The hub-wide rules file on Bolt is shared; a beam's own `_lyx/orch/rules.md` never reaches its parent, since weft content takes no part in a landing merge (`internal/fabricengine` doc).
The readers that today read only the prime's orch read every orch of the hub, the prime's and each beam's: tender's strand and watcher reads for pause, revive and hibernate, `lyx batten overview`'s orch view, and `lyx cleanup`'s orch handoff cleanup.
The orch role stencil tells an orch to read and start only entries whose `parent_beam` is its own worktree, the prime's orch those with an empty one; that filter is how an orch sees only its own tasks.
Rationale: the operator works on a long-lived branch in its own worktree and wants the orch there; keeping batten's shared state at the prime keeps one place every run is read from, and the hub-wide verbs a beam's orch needs act on that place.

### D9. `lyx beam create <slug>`

Decision: the verb runs from the prime and from any beam (D8) and takes an existing project entry which is not done and whose `beam` is false, or whose `beam` is true and whose effective owner (D4) is this machine or none; the project's title, brief and labels are board writing, through `lyx board upsert`, so the verb makes no entry.
It refuses every other entry, each with its way forward: an absent entry or a note names `lyx board upsert` of a project entry under a slug of its own, a task names `lyx beam promote <slug>`, a done project names a new project entry, and a beam another machine owns names the owner and `lyx beam create <slug>` run there, or `lyx adopt <slug>` to take the beam over (D8) and then `lyx beam create <slug>` here.
In order, each step skipped when its result is already there, so a re-run finishes a stopped create:

1. record this machine as the project's owner and set `beam: true`, through the owner setter (D3);
2. create the pair with `Add`, the prime's location acting, `Parent` from `ParentBeam` over the project's own `project`, and the beam mark set in the same origin commit;
3. spawn the pair's push;
4. regenerate the workspace files (D12).

A member started between steps 1 and 2 stops at Worktree-Create with D7's precondition and resumes once the create finishes.
Members already spawned under a grouping project keep the prime as their parent and land there.
The envelope names `lyx orch start` in the beam's worktree as the way to give it an orch; the verb starts none.
Rationale: writing the flag first makes a member started mid-create stop and resume rather than fork from the prime for good; the orch is optional, as the operator decided when beam and orch were split.

### D10. `lyx beam promote <slug>`

Decision: the verb turns a spawned task into a beam, keeping its pair and the commits on its branch.
It refuses, each with its way forward: a task never started, holding no owner and no run status, naming a new project entry through `lyx board upsert` and `lyx beam create` on it; a started task with no pair on this machine, and a task another machine owns, naming `lyx adopt <slug>`; a task whose run has passed Finalize's parent-side merge, naming `lyx batten run <slug>` to finish the landing; and a task an open task depends on, naming `lyx board set-deps`.
In order, each step skipped when done:

1. end the task's run processes and strands, the first part of `lyx batten cancel`'s phase without `--remove`;
2. write the beam mark into the pair's origin record, in one weft commit, pushed;
3. clear the task's claim and batten's records, the rest of that cancel phase;
4. reset the pair's weft content to what a fresh fork from its parent's weft leaves, keeping the pair's origin record, in one weft commit, pushed;
   the reset runs as an executor in `internal/fabricengine/destroy.go` through the shared check pipeline's containment, ownership and dirtiness checks, and first commits the weft's uncommitted record content under `fabricengine.ScopedPathspec`, as `Remove` does before its archive tag, so the weft history keeps it;
   uncommitted weft content outside that pathspec refuses the step, as `Remove`'s no-force sibling-dirt refusal does, and `promote` takes no force flag: the refusal names the paths and, as the way forward, committing or discarding them in the weft worktree and then re-running `lyx beam promote <slug>`;
5. through the owner setter, record this machine as owner and set `beam: true`, and in the same Bolt write turn the entry into a project, clearing `depends_on`, `recipe`, `profile` and `run_settings`;
6. regenerate the workspace files (D12).

Batten's start guard refuses a task whose pair on this machine carries the beam mark in its origin record, naming `lyx beam promote <slug>` to finish the promote, so an entry left an open unclaimed task by a promote stopped after step 3 is never started.
A promote stopped before step 2 leaves the run claimed with its processes ended, as an interrupted cancel does: a resume would restart it, and a re-run of promote ends it again.
The same verb on a project whose `beam` is false is `lyx beam create` on it, and on a project whose `beam` is true and whose pair carries the mark it finishes step 6.
Bound: promote ends a running run for good; nothing of the loom run survives in the pair's checkouts but its warp commits, so no member's weft checkout holds its discussion, plan or run records;
they stay reachable in weft history behind step 4's commit, on the beam's weft branch and on every member's weft branch forked from it.
Rationale: the operator decided a spawned task can become a beam through a verb of its own that releases the run's lock; a member forks its weft from the beam's, so the stopped run's records must not stay there.

### D11. `lyx beam land <slug>` and the `beam` recipe

Decision: the verb refuses, each with its way forward:

- while any open entry's `ParentOf` answer is the beam, through any depth of grouping projects, naming those entries and, for each, landing it, `lyx batten cancel <slug> --remove`, or, for an entry with no pair, moving its `project` off the beam;
- while any local pair's origin record names the beam as its parent, naming each pair and `lyx batten run <slug>` to land it or `lyx batten cancel <slug> --remove`;
- while the beam's orch strand is live or its watcher is, by the liveness `lyx orch status` reports, naming `lyx orch stop` in the beam's worktree and a re-run once `lyx orch status` there reports the watcher gone;
- when run from inside the beam's own worktree, by D8's rule for a verb that destroys a pair, naming the beam's parent worktree as where to run it.

It then starts `lyx batten run <slug> --window` on the project entry, whose window opens in the beam's parent worktree's reed session by D8's rule, with the prime as its working directory.
Batten's start guard applies the first three refusals to a project entry, and Worktree-Create applies them again after its claim, so a resume or a direct `lyx batten run` cannot skip them and a member added before the claim is caught.
The start guard's refusal of a pair left from an earlier run, a run with no status file whose task worktree, records worktree or records branch is present, and its tracked-dirt refusal both give way for a project entry whose `beam` is true and whose local pair carries the beam mark in its origin record, so the landing reaches Worktree-Create, which finds the pair present and skips;
a landing forks no pair, and Finalize's Stuck on a dirty parent pair guards the worktree it merges into.
Every other entry keeps both refusals, bar D7's exemption from the dirt refusal.
A project entry whose `beam` is true and whose local pair has no readable origin record, its records worktree missing, keeps the left-pair refusal, which then names `lyx fabric reconcile` to restore the records worktree and never `lyx batten cancel <slug> --remove`.
`lyx orch start` in a beam whose entry is run-held by its landing refuses, naming the landing, so no orch starts between the claim and the teardown, and a landing leaves no orch strand or watcher of the beam behind.
Batten runs its rows over the beam as over a task: Worktree-Create finds the pair present and skips, Seed-Child seeds recipe `beam` for a project entry, Land prunes the board and deploys when the parent is a deploy branch, and Worktree-Teardown removes the beam pair, whose board settle marks the project done.
The `beam` recipe has two rows: a Go producer that writes the change description from the project's title and brief and the commit subjects the beam adds over its merge base, in the format `summaryparser` validates, and darn's Finalize, which merges the parent in under the parent lock, runs the `finalize` gate and its fixer, and fast-forwards the parent.
The beam's Finalize declares no `on_reroute` and is built so that it never reroutes: a large merge runs the gate like any other, and a spent or over-cap fixer halts for the orch, since shed writes a run failed when a row with no `on_reroute` returns a reroute (`internal/shedengine` doc).
Bound: what this lets through, unseen by any member's review, is the conflict session's resolution of the beam against its parent's newer work, and a textually clean merge of the parent's newer changes to files the beam also changed;
the `finalize` gate over the merged tree alone bounds both, since the fixer's size and kind caps judge only the fixer's change from the HEAD before its first turn, which already holds the resolution;
a fixer change past those caps, or a spent fixer, halts Finalize for the orch.
Bound: a pair on another machine whose origin names the beam is not seen by the refusal when its entry no longer names the beam: a pair made by hand there with `--parent <beam>` and no board entry, and a pair there whose entry's `project` was changed from a machine holding no pair of it, which D2's project-change refusal does not see.
Rationale: the operator decided a beam has no process while it lives and a short landing run at its end; reusing batten and Finalize keeps the owner checks, the parent lock and teardown, and each member was reviewed in its own run, so the landing needs no writer or review.

### D12. Workspace files per beam

Decision: the origin record gains `beam`, true for a beam's pair, written only by `lyx beam create` and `lyx beam promote`.
`RegenerateHubWorkspace` itself changes, so every caller, the fabric verbs and the prime spawn alike, writes, from the origin records:

- the hub's `<repo>.code-workspace`: the prime, Bolt, and every pair whose parent is the prime, beams included, but no pair under a beam;
- one `<repo>-<beam>.code-workspace` per beam, beside it: the beam's worktree, its parent's worktree, Bolt, and every pair whose parent is the beam, a sub-beam by its own worktree only.

A pair whose parent worktree is gone or unrecorded is listed in the hub's file.
Each beam file carries a lyx-generated mark, and the regeneration removes a `<repo>-<slug>.code-workspace` only when it carries the mark and its beam pair is gone, so a landed beam's file goes with its teardown and an operator's own file of that name stays.
Rationale: operator decision of 2026-10-24; the origin record travels with the weft branch, so every machine regenerates the same files without fabric reading the board.

### D13. Parents and notices go to the nearest live orch up the chain

Decision: `hubgeom.ResolveParent` keeps its answer, the pair's recorded parent worktree from its origin record, for every reader that needs the parent pair itself, Finalize's parent-records commit among them.
A new hubgeom function of its own walks from the pair's parent worktree upward through origin records and returns the nearest worktree whose orch strand is live by the engine liveness predicate `lyx orch status` uses, the prime whatever its state when none is.
The notice and parent-name readers take the walk, each resolving at its own call: batten's notices at each delivery, `noticesReachDriversParent`, loom's `primeOrchRecorded` decision, and every caller of `orchcli.NotifyPrime` acting on a run's pair, which gives way to one notify of that run's resolved parent; shuttle's hold notice is one such caller.
A caller acting on no run's pair, such as hibernate's notify and fabric's push notifier, notifies the prime's orch as today; whether a caller acts on a run's pair decides it, never the location it acts from, so batten's notices, sent from the prime's location (D8), take the walk from their run's pair.
The Land row's ready notice takes no walk: it names a newly ready task whose `ParentBeam` is empty to the prime's orch, and one whose `ParentBeam` names a beam to that beam's own orch only when the beam's worktree is present on this machine and its orch strand is live by the same predicate.
Every other ready task, its beam's worktree absent here or its beam's orch stopped or never started, is named in no ready notice, since no orch it would reach may start it: its `parent_beam` keeps it out of every other orch's filter (D8).
Bound: such a task waits unnoticed until the operator or the beam's orch, once started, reads the board; the beam's own orch on another machine gets no notice of it either, as no landing hands work between machines (Out).
Rationale: an orch is optional in a beam and a stopped orch keeps its recorded strand, so only liveness tells where a notice can land; the prime last matches the escalation chain `umbrella-daemon` plans.

### D14. One slug rule, from fabric

Decision: fabric exports its pair slug rule, today's `validateWorktreeSlug` over the hub's configured junction names, and the board applies it, through a seam `boardcli` fills, to every write that makes a task or project slug: an upsert of a new task or project, `lyx board promote` of a note, and a rename.
The refusal names the rule's own reason and the entry, with a slug the rule admits as the way forward.
A write that leaves an entry's slug and kind as they were is never refused by it.
`lyx beam promote`'s kind change to project keeps a slug that already names a pair, so it makes no slug.
A board built without the seam, such as the detached sync, a batten claim or the owner setter, checks nothing; none of them makes a slug.
Rationale: issue #724, where a slug ending in `-weft` reached Worktree-Create; one rule, owned by fabric, keeps the two from drifting.

### D15. README grouping by project

Decision: the README gains `## Projects (n)` before `## Tasks`.
Each open project is an H3 with its handle and title, then its brief, `beam` or `grouping`, and its owner for a beam, then one table of its members in plan order: handle, bold title, kind, and where the member stands (its Running status, Ready, its layer, Note or Done).
Members keep their rows in the Tasks, Notes and Done sections, and a done project is a row of Done.
Rationale: the operator wants a project shown as a plan, with no order prefix in titles.

### D16. `lyx batten cleanup`'s skip names cancel

Decision: `lyx batten cleanup`'s skip of a run that is not done names `lyx batten cancel <slug>` as its way forward.
Rationale: issue #724's blocked-run half is met by `lyx batten cancel`, so the skip points at it.

### Added after Discussion (parent, 2026-10-24): beamcli is a hub-only CLI outside cliwire's standalone wiring

Decision: Card 34 (`beam-create`) states in its Intent that `beamcli` is a hub-only CLI like `adoptcli` and `battencli`, so `PATTERN-cliwire-sole-wiring`'s standalone wiring does not apply: it resolves its location through `battencli.ResolveBattenPrime` (card 23, `batten-guard`) and declares no `cliwire.Module`.

Rationale: F15: only standalone CLIs such as `burlercli` and `webstercli` declare a `cliwire.Module`; hub-only CLIs resolve the prime through battencli, and the card must say which side beamcli is on.

## Constraints

- PATTERN-cwd-resolution: the beam workspace files are hub-level like the hub's file, written directly in the hub dir with hub-inward folders only, never into a worktree; the entry's text changes (see `## PATTERN changes`).
- PATTERN-pair-teardown: its Batten bookend holds, restated: batten's state stays under the prime's anchor, and the process that creates or destroys a pair never runs inside it (D8, D11); the background's "driven from the hub's prime worktree" becomes "anchored at the prime".
- PATTERN-owner-machine: a beam and each of its tasks has its own owner; adopting a beam moves the beam only; an entry with no owner recorded reads as owned by its parent's owner, its parent found through D4's one resolver.
- PATTERN-fabric-git: every git step of D5, D6, D10 and D12 runs inside `internal/fabricengine`, and D10 step 4's weft reset is a destruction executor in `internal/fabricengine/destroy.go`, behind its check pipeline.
- PATTERN-cli-cobra and PATTERN-cliwire-sole-wiring: `lyx beam` is a cobra subtree under the root in `cmd/lyx/main.go`, wired through `internal/cliwire`.
- PATTERN-shed-recipe-registry: the `beam` recipe and its describe producer join `internal/shedrecipe`'s one `Constructor` map.
- PATTERN-tolerant-records: `board.json` and `origin.json` gain keys an older binary keeps as unknown; neither refuses an unknown key. A new `kind` value is not an unknown key: see Open risks.
- PATTERN-refusal-way-forward: every new refusal names its way forward, and a test reaches it.
- PATTERN-stencil-ownership: the orch role stencil's new rule is read at call time through `internal/stencilstore`.
- PATTERN-durable-vs-ephemeral-state: a beam orch's files take the same `_lyx/orch/` and `.lyx/orch/` subpaths in the beam that the prime's take, named only by `internal/lyxdirs`.
- Board neighbours touching the same code: `run-profiles-ladder` (running) changes Seed-Child, which D11 also changes; `batten-window-default` names the prime's reed session for `--window`, which D8 replaces with the run's parent worktree's; `umbrella-daemon`'s `draft` field joins the same `Task` fields and README.
- Docs: `docs/overview.md` gains the `beam` module, and the `ly:board` skill text names the new kind and fields.

## Auto-mode assumptions

- A beam's orch: D8 and D9 (Q7).
- A beam's landing: D11 (Q11).
- Notes as members: D2 (Q3).

## Open risks

- A machine or process on a binary from before this task fails its board replay on a `kind: project` entry, since its `normalizeTask` refuses the kind, and shows a project as a task in its README and layers; every machine of the hub deploys this build before the first project entry is written.
- A run's driver keeps the watched or silent rule it was launched with; a beam orch started or stopped after that launch moves batten's notices but not the driver's own parent message until its next launch.
- Two orchs on one hub share the board and gate slots; nothing beyond D8's stencil filter stops one orch from starting another's task, and the run then lands in the task's beam all the same.
- A hub whose `board.yaml` has a type label beginning with `P` refuses to load its board until the label is renamed.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d10-weft-reset-outside-destroy-chokepoint (class design, severity MEDIUM, open in rounds 3)
  - d13-ready-notice-reaches-barred-prime-orch (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - missing-records-worktree-branch-untested (class scope, severity MEDIUM, open in rounds 2)
  - promote-root-pair-dead-end (class consistency, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

Every refusal below is reached with its way forward in its text.

- D1: an upsert of a project with `depends_on`, `recipe`, `profile`, `run_settings` or `priority` is refused naming the field, as is a write setting a task's `depends_on` to a project, naming its members; a done task whose unchanged `depends_on` names a promoted project takes a later write; a type label beginning with `P` is refused naming the label; a project gets a `P` handle and an owner through the owner setter; an open project with no members lists under layer `Project`, never Ready, and one its landing holds lists as `Running`.
- D2: each refusal of D2's list is reached; a change of `project` on an entry with a local pair is refused naming `lyx batten cancel <slug> --remove`; rename of a beam project is refused naming `lyx board upsert` of its title, and of another entry with a local pair naming `lyx batten cancel <slug> --remove`; marking a project done with members still naming it passes; rename rewrites `project` and `order`; prune drops pruned slugs from `order`, on a run-held project too, and keeps a still-named done project; a merge carrying a member keeps its membership and one removing a still-named project is refused.
- D3: a payload changing `beam` is refused naming `lyx beam create`; one carrying the stored value passes; a hand-set `beam: true` with no branch on origin stops a member at Worktree-Create naming `lyx beam create`, which then makes the pair on the project's effective owner's machine and elsewhere is refused as a beam another machine owns.
- D4: `ParentBeam` returns the nearest open beam through a grouping project, the prime for no project, no beam or only done beams; the one `ParentOf` reads an origin record before the board; batten's start check reads an unspawned beam member as owned by the beam's owner; on the machine holding its pair, `parent_beam` of a member spawned under a grouping project that later became a beam is the prime; a member pair whose beam pair is absent on this machine reads the beam as its `parent_beam` and the beam's owner as its own, and one forked from the prime reads as the prime's on a machine whose prime worktree has another name.
- D5: `Add` with `Parent` forks from the parent's branch and records it in the origin record, with the parent worktree dirty too; a detached parent, a parent on another branch and an unknown parent are refused before any mutation; a member landing into a dirty beam is Stuck at Finalize naming commit or stash in the beam, and resumes once the beam is clean.
- D6: `lyx fabric add --adopt-local` adopts an unmarked local branch, and without the flag it is still refused; batten's `Add` refuses `ErrBranchExists`; a branch checked out elsewhere is refused naming its worktree; the default branch adopted records an empty parent.
- D7: a task whose project is a beam gets a pair whose origin names the beam, whichever worktree starts it; a task of a grouping project gets the prime; an absent beam pair stops Worktree-Create naming `lyx beam create` when its branch is absent from origin and `lyx adopt` when present; a beam run-held by its landing stops it naming the landing; a member of a beam starts and gets its pair while the prime holds tracked dirt.
- D8: `lyx batten run`, `status` and `cancel`, `lyx adopt`, `lyx cleanup` and the `lyx beam` verbs run from a beam worktree acting on the prime, and are refused in a task pair that is not a beam, naming the nearest beam up its origin chain or the prime; `--window` opens in the run's parent worktree's session from any of them; `lyx adopt` of a beam project takes over its owner and adds its pair from origin; `run` and `cancel --remove` from inside the slug's own pair are refused; `lyx orch start` runs in a beam with its state under the beam and its strand `<shortname>:<slug>:orch`, and is refused in a task pair that is not a beam and in a beam another machine owns; tender, overview and cleanup see the beam's orch; a batten verb run from a beam whose entry is run-held by its landing is refused naming the landing and the beam's parent worktree.
- D9: `create` re-run after each step finishes the create, also on a project whose `beam` is already true, and a member started mid-create stops naming `lyx beam create` and later resumes into the beam; an absent entry, a note, a task, a done project and a beam another machine owns are each refused with their way forward.
- D10: `promote` is refused for a task never started, a started task with no local pair, one another machine owns, one past its parent-side merge and one with an open dependent, each with its way forward; it ends the run, leaves the pair's weft holding only its parent's content and the origin record, a promote over an uncommitted weft record file leaving the file reachable in the weft's history, and one over uncommitted weft content outside the record pathspec refused at step 4 naming the paths and a re-run of `lyx beam promote <slug>`, and turns the entry into a beam project; a promote stopped after step 2, 3 or 4 leaves an entry no batten start accepts, naming `lyx beam promote`, and a re-run after any step finishes it.
- D11: `land` is refused with an open member at any depth, a live child pair, a live beam orch strand or watcher, and when run from inside the beam, each with its way forward, and a direct `lyx batten run` of the beam meets the first three; `lyx orch start` in a beam under its landing is refused naming the landing; a landing with a large merge runs the gate without rerouting, merges the beam into its parent, tears it down, leaves no orch strand or watcher of the beam, and marks the project done; a `lyx beam land` of a beam made by `lyx beam create`, and of one made by `lyx beam promote`, passes batten's start guard and reaches Worktree-Create's skip, also with the prime holding tracked dirt, and one whose records worktree is missing is refused naming `lyx fabric reconcile`.
- D12: the hub file lists no pair under a beam, and lists a pair whose parent worktree is gone or unrecorded; a beam file lists its own level only, after a create and after a promote; a prime spawn writes the same files; a landed beam's marked file is removed and an unmarked file of that name stays.
- D13: a run in a beam with a live orch reaches `<shortname>:<beam>:orch`; with no orch, or a stopped one, it reaches the prime's orch; a beam member landing with the beam's orch stopped commits the beam's run records, not the prime's; hibernate's and the push notifier's notices reach the prime's orch; a ready task of a beam is named to that beam's orch while it is live, one whose beam's worktree is absent on this machine is named to none, and a ready task of a beam whose orch is stopped or never started is named to the prime's orch not at all.
- D14: a new task or project slug ending in `-weft` or equal to a junction name is refused at upsert, at `lyx board promote` of a note and at rename with fabric's reason; an existing such slug left unchanged is not, and `lyx beam promote` checks none.
- D15: the Projects section lists members in plan order with their standing, unordered members after by handle.
- D16: `lyx batten cleanup`'s skip of a run that is not done names `lyx batten cancel <slug>`.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd or worktree root: only `internal/lyxcwd`; an engine is handed absolute paths and never imports it; hub-level containers and the hub's workspace files stay reachable from the hub only.
- `PATTERN-pair-parent` — adds: Picking a new pair's parent for a board entry or reading an entry's owner through its parent: only `ParentBeam` and hubgeom's one `ParentOf`, never the acting worktree.
