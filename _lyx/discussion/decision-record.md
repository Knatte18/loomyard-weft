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
- Batten, `lyx adopt` and `lyx cleanup` run from any drivable worktree, with batten's state kept under the prime; Worktree-Create forks from the task's beam.
- Notices and parent names go to the nearest live orch up the beam chain.
- Orch: `lyx orch` runs from the prime and from any beam, one orch per worktree, named `ly:<slug>:orch` in a beam.
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
It carries no `depends_on`, `recipe`, `profile`, `run_settings` or `priority`, and a write giving it one is refused.
No task may depend on a project, as no task may depend on a note today.
Its handle letter is `P`; `LoadConfig` refuses a type label whose letter is `P`, as it refuses `T`, naming the label and renaming it in `board.yaml` as the way forward.
Only a task is claimable, bar the landing run of a project whose `beam` is true (D11).
A project carries `owner_machine` as a task does: the owner setter, `Owners.Effective` and the list view's owner treat a project like a task.
Rationale: the operator decided a project entry on the board; promote (D10) needs a kind change, so a flag on a task cannot carry it.

### D2. Membership: `project` on an entry, plan order on the project

Decision: a task and a note may carry `project`, the slug of a project entry; a project's `order` lists member slugs in plan order.
A write is refused when, for an entry whose `project` or `order` it sets or changes, it leaves:

- a `project` naming an absent entry, a non-project entry, the entry itself, a done project, a project whose status is a run status, or a project chain with a cycle;
- an `order` naming an entry whose `project` is not this project, or naming one entry twice.

An entry whose `project` the write leaves as it was is never refused by these rules, so marking a project done, by hand or at its landing, refuses nothing.
A member missing from `order` renders after the ordered ones, by handle.
Promote keeps a note's `project`.
Rename rewrites every `project` and `order` naming the old slug.
Prune drops pruned slugs from every `order`, and keeps a done project while an entry the same prune does not remove names it.
The run lock lets prune's `order` rewrite through on a held project, as it lets prune's `depends_on` rewrite through today.
A merge carries the removed entries' memberships to the upserted entry and refuses to remove a project another entry still names.
`project`, `order` and `beam` are otherwise scope, so the run lock refuses a change to them on a run-held entry.
Rationale: the operator decided both the per-entry field and the project's ordered list; notes are members because the `workspaces` project holds notes such as `umbrella-daemon` and `umbrella-anchor`.

### D3. The `beam` flag is written only by the beam verbs

Decision: an upsert, batch or merge payload may carry `beam` only as the entry's stored value, dropped as a no-op; any other value is refused naming `lyx beam create` or `lyx beam promote`.
`boardengine`'s owner setter gains the beam flag: `lyx beam create` and `lyx beam promote` record this machine as owner and set `beam: true` in its one Bolt commit, and are its only callers that pass the flag.
Nothing sets it back: a landed beam is a done project.
Bound: a hand edit of `board.json` can still set it; batten then finds no beam pair and stops at Worktree-Create with D7's precondition.
Rationale: a `beam: true` set without the verb would send every member to a pair that may never be made.

### D4. One parent resolver: origin record, else the project chain

Decision: `ParentBeam` in `boardengine` walks an entry's `project` links upward, skipping done projects, and returns the nearest project whose `beam` is true, or the empty slug for the prime.
One `ParentOf` built in `internal/hubgeom` is the only one every owner reader is handed, batten's, adopt's, loom's owner check, fabric remove's, pair teardown's and `boardcli`'s alike: it reads the slug's origin record and, only when no record is found, `ParentBeam` over the local board.
Origin found with an empty parent reads as the prime and never falls through to the board.
Batten's Worktree-Create (D7) and `lyx beam create` (D9) use `ParentBeam` to pick a new pair's parent.
`lyx board list` and `get` carry each entry's `ParentBeam` as `parent_beam`, empty for the prime.
A grouping project, one whose `beam` is false, inside a beam passes its members on to that beam.
Rationale: the operator decided that a task of a beam forks from and lands into the beam, a task whose project has no worktree goes to the prime, and every beam and task has its own owner with an unrecorded owner read from the parent; one resolver keeps the start, claim, landing and teardown verdicts alike.

### D5. Fabric creates a pair from a named parent pair

Decision: `AddOptions.Parent` names a pair slug of the hub; empty keeps today's fork from the acting worktree's HEAD.
Every caller that picks a parent from the board passes the prime's location as the acting location, so an empty `ParentBeam` forks from the prime wherever the caller runs.
With a parent named, `Add` forks the warp branch from the parent worktree's HEAD and the weft branch from the parent's weft branch, and records the parent's branch as `ParentBranch` and its worktree name as `ParentWorktree`.
It refuses, before any mutation, a parent that is not a pair of this hub, a parent worktree on a detached HEAD, and one whose checked-out branch is not its own pair branch, naming `lyx fabric checkout` as the way forward.
It does not refuse tracked dirt in a named parent's worktree: the pair forks from the parent's last commit, and the dirty paths are logged at Info.
`lyx fabric add <slug> --parent <slug>` exposes it.
Merge-back, the parent lock and teardown need no change: they already follow the origin record.
Bound: uncommitted edits in a beam never reach its members, and starting a member never waits on them.
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
The beam's pair absent on this machine stops the row with a `PreconditionError` naming the beam and its way forward: `lyx beam create <beam>` when the board names no owner for it, else `lyx adopt <beam>`.
A beam whose board entry is run-held by its own landing stops the row with a `PreconditionError` naming the landing.
Batten's start guard keeps its tracked-dirt refusal for a pair forked from the prime, and checks none for one forked from a beam (D5).
Rationale: the operator decided that a project with a worktree is where its tasks spawn from and land into; the origin record then carries the rest.

### D8. Batten, adopt, cleanup and orch from any drivable worktree

Decision, batten, adopt and cleanup: `RefuseUnlessPrime` gives way, for every batten verb, `lyx adopt` and `lyx cleanup`, to a guard that admits any drivable worktree of the hub and resolves the prime's location from it; each verb then acts on the prime's location as it does today.
Every batten path, the status file, locks, scratch and the prime lock, stays under the prime's anchor.
A verb that creates or destroys a pair refuses when the acting worktree is that pair itself, naming the parent or the prime as where to run it.
`lyx batten run --window` opens its window in the acting worktree's reed session with the prime as its working directory.
Decision, orch: every `lyx orch` verb runs in the prime and in a beam, a pair whose origin record carries the beam mark (D12), and refuses elsewhere naming `lyx beam create` or `lyx beam promote`.
A beam's orch keeps its files under the beam's own `.lyx/orch/` and `_lyx/orch/`, runs in the beam pair's reed session, and is the strand `ly:<slug>:orch`, the name `hubgeom.ResolveParent` already gives a pair forked from that worktree; its watcher is its own.
`lyx orch start` in a beam another machine owns refuses, naming the owner and `lyx adopt <slug>`.
The hub-wide rules file on Bolt is shared; a beam's own `_lyx/orch/rules.md` never reaches its parent, since weft content takes no part in a landing merge (`internal/fabricengine` doc).
The readers that today read only the prime's orch read every orch of the hub, the prime's and each beam's: tender's strand and watcher reads for pause, revive and hibernate, `lyx batten overview`'s orch view, and `lyx cleanup`'s orch handoff cleanup.
The orch role stencil tells an orch to read and start only entries whose `parent_beam` is its own worktree, the prime's orch those with an empty one; that filter is how an orch sees only its own tasks.
Rationale: the operator works on a long-lived branch in its own worktree and wants the orch there; keeping batten's shared state at the prime keeps one place every run is read from, and the hub-wide verbs a beam's orch needs act on that place.

### D9. `lyx beam create <slug>`

Decision: the verb runs from any drivable worktree and takes an existing project entry whose `beam` is false and which is not done; the project's title, brief and labels are board writing, through `lyx board upsert`, so the verb makes no entry.
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
It refuses a task with no pair on this machine, a task another machine owns, a task whose run has passed Finalize's parent-side merge, and a task an open task depends on, naming `lyx board set-deps`.
In order, each step skipped when done:

1. end the task's batten run through `lyx batten cancel`'s phase without `--remove`, which ends its processes and strands and clears its claim and batten's records;
2. reset the pair's weft content to what a fresh fork from its parent's weft leaves, keeping the pair's origin record, and write the beam mark into that record, in one weft commit, pushed;
3. through the owner setter, record this machine as owner and set `beam: true`, and in the same Bolt write turn the entry into a project, clearing `depends_on`, `recipe`, `profile` and `run_settings`;
4. regenerate the workspace files (D12).

The same verb on a project whose `beam` is false is `lyx beam create` on it.
Bound: promote ends a running run for good; nothing of the loom run survives but its warp commits, so no member's weft inherits its discussion, plan or run records.
Rationale: the operator decided a spawned task can become a beam through a verb of its own that releases the run's lock; a member forks its weft from the beam's, so the stopped run's records must not stay there.

### D11. `lyx beam land <slug>` and the `beam` recipe

Decision: the verb refuses while any open entry's `ParentBeam` is the beam, through any depth of grouping projects, or any local pair's origin record names the beam as its parent; it then starts `lyx batten run <slug>` on the project entry, in its own window in the parent worktree's reed session with the prime as its working directory.
Batten's start guard applies the same refusal to a project entry, and Worktree-Create applies it again after its claim, so a resume cannot skip it and a member added before the claim is caught.
Batten runs its rows over the beam as over a task: Worktree-Create finds the pair present and skips, Seed-Child seeds recipe `beam` for a project entry, Land prunes the board and deploys when the parent is a deploy branch, and Worktree-Teardown removes the beam pair, whose board settle marks the project done.
The `beam` recipe has two rows: a Go producer that writes the change description from the project's title and brief and the commit subjects the beam adds over its merge base, in the format `summaryparser` validates, and darn's Finalize, which merges the parent in under the parent lock, runs the `finalize` gate and its fixer, and fast-forwards the parent.
The beam's Finalize declares no `on_reroute` and is built so that it never reroutes: a large merge runs the gate like any other, and a spent or over-cap fixer halts for the orch, since shed writes a run failed when a row with no `on_reroute` returns a reroute (`internal/shedengine` doc).
Bound: a pair made by hand on another machine with `--parent <beam>` and no board entry is not seen by the refusal.
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

Decision: `hubgeom.ResolveParent` walks from the pair's parent worktree upward through origin records and returns the nearest worktree whose orch strand is live by the engine liveness predicate `lyx orch status` uses, the prime whatever its state when none is.
Every reader resolves at its own call: batten's notices at each delivery, `noticesReachDriversParent`, loom's `primeOrchRecorded` decision, and every caller of `orchcli.NotifyPrime`, which gives way to one notify of a run's resolved parent; shuttle's hold notice is one such caller.
The Land row's ready notice names each newly ready task to the orch the same walk finds from the worktree of that task's `ParentBeam`, the prime's for an empty one.
Rationale: an orch is optional in a beam and a stopped orch keeps its recorded strand, so only liveness tells where a notice can land; the prime last matches the escalation chain `umbrella-daemon` plans.

### D14. One slug rule, from fabric

Decision: fabric exports its pair slug rule, today's `validateWorktreeSlug` over the hub's configured junction names, and the board applies it, through a seam `boardcli` fills, to every write that makes a task or project slug: an upsert of a new task or project, a promote, a rename, and a kind change to project.
The refusal names the rule's own reason and the entry; a note's slug is checked when it is promoted.
A write that leaves an entry's slug and kind as they were is never refused by it.
A board built without the seam, such as the detached sync, a batten claim or the owner setter, checks nothing; none of them makes a slug.
`lyx batten cleanup`'s skip of a run that is not done names `lyx batten cancel <slug>` as its way forward.
Rationale: issue #724, where a slug ending in `-weft` reached Worktree-Create; one rule, owned by fabric, keeps the two from drifting.

### D15. README grouping by project

Decision: the README gains `## Projects (n)` before `## Tasks`.
Each open project is an H3 with its handle and title, then its brief, `beam` or `grouping`, and its owner for a beam, then one table of its members in plan order: handle, bold title, kind, and where the member stands (its Running status, Ready, its layer, Note or Done).
Members keep their rows in the Tasks, Notes and Done sections, and a done project is a row of Done.
Rationale: the operator wants a project shown as a plan, with no order prefix in titles.

## Constraints

- PATTERN-cwd-resolution: the beam workspace files are hub-level like the hub's file, written directly in the hub dir with hub-inward folders only, never into a worktree; the entry's text changes (see `## PATTERN changes`).
- PATTERN-pair-teardown: its Batten bookend holds, restated: batten's state stays under the prime's anchor, and the process that creates or destroys a pair never runs inside it (D8, D11); the background's "driven from the hub's prime worktree" becomes "anchored at the prime".
- PATTERN-owner-machine: a beam and each of its tasks has its own owner; adopting a beam moves the beam only; an entry with no owner recorded reads as owned by its parent's owner, its parent found through D4's one resolver.
- PATTERN-fabric-git: every git step of D5, D6, D10 and D12 runs inside `internal/fabricengine`.
- PATTERN-cli-cobra and PATTERN-cliwire-sole-wiring: `lyx beam` is a cobra subtree under the root in `cmd/lyx/main.go`, wired through `internal/cliwire`.
- PATTERN-shed-recipe-registry: the `beam` recipe and its describe producer join `internal/shedrecipe`'s one `Constructor` map.
- PATTERN-tolerant-records: `board.json` and `origin.json` gain keys an older binary keeps as unknown; neither refuses an unknown key. A new `kind` value is not an unknown key: see Open risks.
- PATTERN-refusal-way-forward: every new refusal names its way forward, and a test reaches it.
- PATTERN-stencil-ownership: the orch role stencil's new rule is read at call time through `internal/stencilstore`.
- PATTERN-durable-vs-ephemeral-state: a beam orch's files take the same `_lyx/orch/` and `.lyx/orch/` subpaths in the beam that the prime's take, named only by `internal/lyxdirs`.
- Board neighbours touching the same code: `run-profiles-ladder` (running) changes Seed-Child, which D11 also changes; `batten-window-default` names the prime's reed session for `--window`, which D8 replaces with the acting worktree's; `umbrella-daemon`'s `draft` field joins the same `Task` fields and README.
- Docs: `docs/overview.md` gains the `beam` module, and the `ly:board` skill text names the new kind and fields.

## Auto-mode assumptions

- The orch thread of a beam is the orchestrator, run through `lyx orch` (the board entry's orch recommendation), and the beam verbs start none.
- The landing of a beam needs no writer and no review.
- Notes may be project members.

## Open risks

- A machine or process on a binary from before this task fails its board replay on a `kind: project` entry, since its `normalizeTask` refuses the kind, and shows a project as a task in its README and layers; every machine of the hub deploys this build before the first project entry is written.
- A run's driver keeps the watched or silent rule it was launched with; a beam orch started or stopped after that launch moves batten's notices but not the driver's own parent message until its next launch.
- Two orchs on one hub share the board and gate slots; nothing beyond D8's stencil filter stops one orch from starting another's task, and the run then lands in the task's beam all the same.
- A hub whose `board.yaml` has a type label beginning with `P` refuses to load its board until the label is renamed.

## Acceptance criteria

- Board: an upsert of a project with `depends_on`, `recipe`, `profile`, `run_settings` or `priority` is refused, as is a task depending on a project; a type label beginning with `P` is refused naming the label; a project gets a `P` handle and an owner through the owner setter.
- Board: each refusal of D2 is reached; marking a project done with members still naming it passes; rename rewrites `project` and `order`; prune drops pruned slugs from `order`, on a run-held project too, and keeps a still-named done project; a merge carrying a member keeps its membership.
- Board: a payload changing `beam` is refused naming `lyx beam create`; one carrying the stored value passes.
- Board: `ParentBeam` returns the nearest open beam through a grouping project, the prime for no project, no beam or only done beams; the one `ParentOf` reads an origin record before the board, and batten's start check reads an unspawned beam member as owned by the beam's owner.
- Board: a new task slug ending in `-weft` or equal to a junction name is refused at upsert, promote and rename with fabric's reason; an existing such slug left unchanged is not.
- README: the Projects section lists members in plan order with their standing, unordered members after by handle.
- Fabric: `Add` with `Parent` forks from the parent's branch and records it in the origin record, with the parent worktree dirty too; a detached parent, a parent on another branch and an unknown parent are refused before any mutation.
- Fabric: `lyx fabric add --adopt-local` adopts an unmarked local branch, and without the flag it is still refused; batten's `Add` refuses `ErrBranchExists`; a branch checked out elsewhere is refused naming its worktree; the default branch adopted records an empty parent.
- Batten: a task whose project is a beam gets a pair whose origin names the beam, whichever worktree starts it; a task of a grouping project gets the prime; an absent beam pair stops Worktree-Create with its precondition.
- Batten: `lyx batten run`, `status` and `cancel`, `lyx adopt` and `lyx cleanup` run from a beam worktree acting on the prime; `run` and `cancel --remove` from inside the slug's own pair are refused.
- Orch: `lyx orch start` runs in a beam with its state under the beam and its strand `ly:<slug>:orch`; it is refused in a task pair that is not a beam and in a beam another machine owns; tender, overview and cleanup see the beam's orch.
- Notices: a run in a beam with a live orch reaches `ly:<beam>:orch`; with no orch, or a stopped one, it reaches the prime's orch; a ready task of a beam is named to that beam's orch.
- Beam: `create` re-run after each step finishes the create, and a member started mid-create stops and later resumes into the beam; `promote` is refused with a dependent, ends the run, leaves the pair's weft holding only its parent's content and the origin record, and turns the entry into a beam project; `land` is refused with an open member at any depth or a live child pair; a landing with a large merge runs the gate without rerouting, merges the beam into its parent, tears it down and marks the project done.
- Workspace: the hub file lists no pair under a beam; a beam file lists its own level only, after a create and after a promote; a prime spawn writes the same files; a landed beam's marked file is removed and an unmarked file of that name stays.

## PATTERN changes

- `PATTERN-cwd-resolution` — changes: Resolving a cwd or worktree root: only `internal/lyxcwd`; an engine is handed absolute paths and never imports it; hub-level containers and the hub's workspace files stay reachable from the hub only.
- `PATTERN-pair-parent` — adds: Creating a pair for a board entry or reading its parent: only `ParentBeam` and hubgeom's one `ParentOf`, never the acting worktree.
