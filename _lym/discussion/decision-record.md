# Decision record — testmap-checked-in

## Goal

Move the coverage maps out of the never-tracked `<bolt>/.lym/testcover/` and into git, so that a map follows the branch whose code it describes.
Settle where they live, how a new pair gets them, what they cost in git and on disk, how machines share them, who commits them, and whether the upkeep pass runs by default.

## Scope

In: D1 to D11.

Out:

- The selector's miss rules (board note `selector-non-body-miss`).
- Cutting tests (board note `redundancy-sweep-existing`).
- Measuring upkeep cost, a state view, and the operator guide (task `testmap-upkeep-cost-and-guide`).
- A new branch, worktree or junction in the weft repository.
- Delivering a task's maps to its parent by any merge.

## Decisions

### D1 — The maps live in the pair's own weft branch, under `_lym/testcover/`

A worktree's maps sit at `<anchor>/_lym/testcover/<package>/map.json`, tracked on that pair's weft branch.
`internal/lymdirs` names the directory, keeping `lymdirs.TestCoverDirName`; `testcover.Dir` takes the worktree's anchor in place of the bolt.
Every reader and writer works on the maps of the worktree it runs in: the upkeep pass, the gate's coverage scope, `lym gate redundant` and the audit.
The design adds no new branch, worktree or junction to the weft repository.
Rationale: the operator's direction (2026-11-12) is that a task keeps its own maps, they are not delivered, and a new weft starts from the parent's; the existing weft branch does all three (see Constraints).

### D2 — A new pair inherits its parent's committed maps at fork

The weft fork keeps `_lym/testcover/` from the parent's weft tip, as it keeps everything but run records today.
The child gets the maps the parent last committed; a map the parent wrote but has not committed is not inherited.
A pair forked before this change, or before the prime holds maps, has none: its gate reads `testcover.MissNoMap` and runs whole packages until a pass in it builds maps.

### D3 — A landing reaches the parent's maps through the parent's upkeep pass

When a task lands, its maps stay on its weft branch and are archived with it by the teardown's archive tag.
The parent's next upkeep pass plans every map at the parent's new HEAD, as `testcover.PlanUpkeep` does today: it keeps every entry but those the landed diff affects, and re-measures those.
Two tasks that touch one package never conflict on a map file, since neither task's map file reaches the parent.

### D4 — `lym gate testmap` commits and pushes the maps it changed

Both modes of the verb, the upkeep pass and the rebuild of named packages, end by committing every changed path under the worktree's `_lym/testcover/`.
The verb writes the maps into the weft worktree's own `_lym` path, which `fabricengine` hands it, never through the warp-side junction.
It commits them through `fabricengine`'s scoped weft commit (`internal/fabricengine/commitweftpaths.go`), with a positive pathspec of that directory only, under fabric's records write lock like any records commit.
It then spawns the worktree's detached records push: records-only in the prime, and with the code branch in a task pair, per `PATTERN-pair-remote`.
A killed or failed call leaves its written maps uncommitted.
The next call commits them; so may any other records commit of the pair whose pathspec covers `_lym`, such as a teardown's pending-records commit, which is harmless since each map file is written whole.
The exit record and the build locks stay never-tracked under the worktree's `.lym/testcover/`, through a scratch accessor beside `testcover.Dir`, and the dobby reads the prime's exit record there.
Each map's lock is the never-tracked one `lock.TrackedFileLockPath` gives, under the checkout's `.lym/locks/` mirror.
Bound: the verb's own commit covers only `_lym/testcover/`; it never commits another path of the weft.

### D5 — Uncommitted maps do not make a worktree dirty for preflight

`preflight.Check`'s worktree-clean check exempts `_lym/testcover/`, beside the run-records root it exempts today (`internal/preflight/preflight.go`).
Rationale: D4 leaves maps uncommitted while a call runs, or after a killed call until the next; without the exemption every tier-2 orchestrator in that worktree refuses meanwhile.
Bound: the exemption covers that subtree alone; a change anywhere else under `_lym` stays dirty.

### D6 — Only the owner machine writes maps; the stamp stays per environment

`lym gate testmap` refuses on a machine that reads the worktree's entry as owned by another machine, read through `hubgeom`'s `ParentOf` as for a run.
The refusal names the owner machine and the way forward: run it there.
`testcover.EnvironmentStamp` keeps its fields as they are, and two machines read each other's maps as current only when no machine-layer env value differs between them and the reader's store holds the map's commit.
The hub env digest hashes envconfig's stacked `vars`, the `env.machine.yaml` machine layer included (`internal/envconfig/config.go`, where `vars` is `CategoryMachine`), so a differing machine-layer variable makes every map a `testcover.MissEnvironment` on the other machine.
A reader whose code store lacks a map's commit, such as an unpushed prime HEAD, reads `testcover.MissCommitUnreadable` for that map.
Both cases degrade to whole-package runs and never to a wrong selection.
A non-owner machine whose stamp differs reads every map as a `testcover.MissEnvironment` and runs whole packages, and never rewrites the maps.
An owner machine whose stamp changes, as after a Go upgrade, rebuilds and commits every map in its next passes.
Bound: a hub with no owner recorded for the entry reads as owned by no other machine, so two machines can both commit its maps and meet a conflict on the weft pull; recording the owner with `lym adopt` closes it.

### D7 — A pass writes a map only when its content changes

A map whose plan is `UpkeepCurrent` and whose entries, queue and failure record are unchanged by the plan keeps its file as it is, its `commit` included.
The next pass plans the map from that older commit against the newer HEAD.
Every other map is written as today, its `commit` and format commit moving together.
A map's `commit` now means the commit its content was last written at; the exit record's `planned_head` says which HEAD every map was last planned at.
Every reader that compares a map's `commit` with HEAD to mean the map is current reads the planned HEAD instead.
Inside a pass that is the pass's own HEAD, at which every map it planned without a miss is current;
the audit's candidate filter (`internal/gatecli/testmapaudit.go`) and the guard of a package job that runs tests into the planned map (`internal/gatecli/testmappackage.go`) are two such readers.
Outside a pass it is the exit record's `planned_head`.
A pass that runs a test into a map it left unwritten, as the audit does, writes the planned map at HEAD with the fresh entry, so its `commit` and file hashes move to HEAD.
Rationale: with D7 a landing commits only the maps it changed.
Bound: the commit a map stays at must stay reachable.
The `internal/fabricengine` package doc says a code branch is never rebased, and a rewind of the prime's code branch resets as a rewrite does.
A fetch never prunes a dropped commit's object, so it stays readable until a teardown's `git gc --auto` prunes it.
A map whose commit is gone from the store reads `testcover.MissCommitUnreadable`, which an upkeep pass plans as `UpkeepMiss` and rebuilds whole at HEAD;
the bound therefore degrades to one rebuild and never to a wrong selection.

### D8 — The map file is compact JSON

The map file is written with no indentation, its field order as today.
`internal/state`'s JSON writer gains a compact form the map writer uses, keeping the unknown-key carry-over `PATTERN-tolerant-records` requires.
The reader accepts the indented form too, so the format version stays 2.
Rationale: the indentation puts one integer per line in each block list; compaction cuts the checkout of every weft worktree by about a quarter.

### D9 — `lym gate testmap` runs in the prime or a task worktree

The verb runs in any worktree of a hub that has a weft, on that worktree's own maps at its own HEAD.
It refuses outside a hub, as today, naming the way forward.
The prime's dobby starts the pass automatically in the prime only, as today; in a task, the operator or an agent runs it by hand.
A hand rebuild in a task, such as the one the redundancy sweep runs before judging a package, commits to that task's weft only and never reaches the parent.
A pass registers in `internal/runproc` as `runproc.RoleTestMap` under its worktree's owner: `runproc.HubOwner` in the prime, as today, and the task's slug in a task.
A pair's teardown ends the processes registered under its slug before the worktree goes, so it ends a task's pass, per `PATTERN-child-processes`.
Bound: the one-pass-at-a-time refusal and the dobby's `TestMapRunning` read count only passes of the same worktree, so a task's pass neither blocks nor is refused by the prime's.

### D10 — `test_map_upkeep` is off by default

`internal/dobbyengine/template.yaml` sets `test_map_upkeep: false`.
The operator turns it on with `lym config` in the prime.
A layer that holds the key keeps its value, and a prime whose layers never set it reads the new default, so its dobby stops starting passes.
Rationale: the operator did not authorise the first 44-minute build, and a build that commits to git is the operator's call.
This is a default change, not a change of the key's meaning, so no `configreg.Reinterpretation` ships.

### D11 — One import of the existing maps into the prime's weft

A call in the prime first removes from `<bolt>/.lym/testcover/` each old `map.json`, with its locks, whose package has a committed map in the prime's weft; it does so on any machine, before D6's owner refusal.
On the owner machine it then imports each old `map.json` whose package has no map in the prime's `_lym/testcover/`, writing it there in D8's compact form, plans as usual, commits per D4, and runs the removal again.
D4's commit takes every imported map, whether or not the pass plans its package, so each old map gets a committed counterpart and is removed.
D6's refusal comes before the import, so a non-owner machine never writes a map;
its own old bolt maps are removed by its prime calls once its pulled weft holds a committed map for their package.
It keeps the old `testmap-exit.json`, which `testmap-upkeep-cost-and-guide` reads as a pass record.
The operator makes the first prime call after the landing, by hand with `lym gate testmap` in the prime, since D10 stops the dobby's automatic pass; Open risks states the window until then.
Bound: it runs in the prime only, imports only packages the prime holds no map for, and removes an old map only once the prime holds a committed map for its package.
A call after an import killed partway therefore imports the rest and removes none of them before they are committed.
Rationale: it saves a 44-minute rebuild of maps that are current at `79f0eb4a`.

### Added after Discussion (arbiter, 2026-11-12): D1 stands: the maps use the pair's existing weft branch, with no extra branch or junction

Decision: D1 is confirmed: the maps live on each pair's existing weft branch under _lym/testcover/, and the task adds no extra branch, worktree or junction to the weft repository, as Scope Out already says. The fixer drops the Open-risks line that begins 'Decision for the parent: the board entry's Idea names one extra branch and one extra junction' and cites this decision in D1's rationale.

Rationale: The board entry's Idea gives the extra branch one purpose: holding lym data that is meant to be merged, apart from the discarded records. The operator's later Direction (2026-11-12, thought through) says a task's maps are not delivered: the task keeps its own maps in its own weft, the parent's upkeep pass carries the parent's maps forward, and a new pair's weft starts from the parent's weft. The existing weft branch does all three (fork in internal/fabricengine/forkrecords.go keeps everything but run records), so no map needs the merge the extra branch was for. Confirming D1 narrows the task and does not widen it. Any other mergeable lym data is outside this task.

## Constraints

- `PATTERN-lym-files`: the tracked maps go under `_lym`, the never-tracked exit record and build locks under `.lym`, at the mirrored subpath, and each map's lock under the checkout's `.lym/locks/`; only `internal/lymdirs` names either.
- `PATTERN-child-processes`: D9 registers a task's pass under the task's slug, so its pair's teardown ends it.
- `PATTERN-fabric-boundary`: the weft commit and push go through `internal/fabricengine` with a positive-only pathspec; D4's verb writes into the weft path fabric hands it and commits at its own end, never through the junction.
- `PATTERN-pair-remote`: a task pair's records push carries its code branch; the prime's stays records-only.
- `PATTERN-owner-machine`: see PATTERN changes.
- `PATTERN-tolerant-records`: the map reader keeps accepting unknown keys; D8 changes no key.
- `PATTERN-refusal-way-forward`: D6's and D9's refusals name the way forward.
- Fact (code at `79f0eb4a`): a pair's weft branch forks from its parent's weft branch and drops only shed run records (`internal/fabricengine/forkrecords.go`); weft branches never merge, and teardown archives a branch's tip with a tag.
- Fact (code at `79f0eb4a`): `testcover.PlanUpkeep` sets `Changed` whenever the map's commit differs from the new commit, so a pass rewrites every current map today.
- Fact (code at `79f0eb4a`): the selector misses with `MissUnmarked` when a map's format commit differs from its commit, and compares the map's recorded file hashes against the target's (`internal/testcover/testselect.go`).
- Fact (code at `79f0eb4a`): the hub env digest hashes envconfig's stacked `vars`, the `env.machine.yaml` machine layer included, and the tiers digest the mapped tiers' commands and env (`internal/gatecli/testmappackage.go`); neither holds a machine name, but a machine-layer value can differ per machine.

## Auto-mode assumptions

- A task does not need its maps carried to its own HEAD for its gate; the selector reads the inherited maps against the task's tree.
- The prime's dobby stays the only automatic starter; a beam parent's maps are kept by hand as a task's are.

## Open risks

- Decision for the parent: the board entry's Idea names one extra branch and one extra junction in the weft repository; D1 puts the maps on the existing weft branch instead, because D3 delivers no map by merge. Confirm, or record the extra branch with `lym loom decision add`.
- Decision for the parent: the entry's Idea says raddle splits the work into several tasks because of the new branches; with D1 there is no new branch, and the work may fit one task.
- `testmap-upkeep-cost-and-guide` reads the maps' place, the upkeep default and "which maps are at HEAD"; D1, D7 and D10 change all three, so its guide is right only if it runs after this task lands or follows this record.
- From the landing until the operator's first `lym gate testmap` in the prime, every gate reads `testcover.MissNoMap` and runs whole packages, and every new pair inherits no maps.
- Each live weft worktree checks out the whole map set, about 41 MB compact; twenty pairs hold about 0.8 GB.
- The prime's weft history grows by the maps each landing changes; the stores never auto-gc, and only a pair teardown runs `git gc --auto`.

## Acceptance criteria

- D1: the upkeep pass, the gate's coverage scope and `lym gate redundant` read and write the maps under the worktree's own `_lym/testcover/`, and none reads `<bolt>/.lym/testcover/` but D11's import.
- D2: a pair added from a parent whose weft holds committed maps has the same maps in its own `_lym/testcover/`.
- D3: after a task lands, the parent's pass rebuilds only the affected tests' entries, and the task's map files do not appear on the parent's weft branch.
- D4: an upkeep pass and a named-package rebuild that changed maps each leave one weft commit holding only paths under `_lym/testcover/`, and the exit record and locks are untracked.
- D4: a killed call's written maps are committed by the next call.
- D5: a worktree whose only uncommitted change is under `_lym/testcover/` passes preflight's clean check, and one with another uncommitted `_lym` change fails it.
- D6: `lym gate testmap` on a machine that reads the entry as owned by another refuses, naming the owner and the way forward, and commits nothing.
- D7: a pass at a HEAD that affects no test of a package leaves that package's `map.json` byte-for-byte unchanged, and the next pass still plans it as current.
- D7: a pass that leaves a package's map unwritten still lets the audit sample that package's tests.
- D8: a written map is compact JSON, and an indented map is still read.
- D9: `lym gate testmap` in a task worktree plans and writes that task's maps at the task's HEAD.
- D9: a pass in a task does not block the prime's dobby pass, and its pair's teardown ends it.
- D10: a prime whose layers hold no `test_map_upkeep` key gets no automatic pass from its dobby; one that sets it true gets one as today.
- D11: a prime call with an empty `_lym/testcover/` and a populated `<bolt>/.lym/testcover/` imports the maps compact, commits them, and removes the old maps and locks but not the old exit record; a second call imports nothing.
- D11: a call after an import killed partway imports the rest and removes no old map before its package's map is committed.
- D11: a prime call on a non-owner machine imports nothing, and removes only old maps whose package has a committed map in its weft.
- D11: a prime call whose maps are committed removes old maps still left in the bolt's directory.
- Review: `pattern/PATTERN-test-speed.md` and the `internal/testcover` and `internal/gatecli` package docs describe the maps per worktree in the weft, not in the bolt's `.lym`.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, committing coverage maps, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.

## Notes for the plan writer

- `testcover.Dir` and the callers that pass `fabricengine.BoltDir` are where D1 turns; `internal/gatecli/testmap.go` holds the prime-only refusal D9 widens.
