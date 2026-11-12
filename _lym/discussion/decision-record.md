# Decision record — testmap-checked-in

## Goal

Move the coverage maps out of the never-tracked `<bolt>/.lym/testcover/` and into git, so that a map follows the branch whose code it describes.
Settle where they live, how a new pair gets them, what they cost in git and on disk, how machines share them, who commits them, and whether the upkeep pass runs by default.

## Scope

In: D1 to D10.

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
The exit record, the locks and the build locks stay never-tracked, under the worktree's `.lym/testcover/`, and the dobby reads the prime's exit record there.
Bound: the verb's own commit covers only `_lym/testcover/`; it never commits another path of the weft.

### D5 — Uncommitted maps do not make a worktree dirty for preflight

`preflight.Check`'s worktree-clean check exempts `_lym/testcover/`, beside the run-records root it exempts today (`internal/preflight/preflight.go`).
Rationale: D4 leaves maps uncommitted while a call runs, or after a killed call until the next; without the exemption every tier-2 orchestrator in that worktree refuses meanwhile.
Bound: the exemption covers that subtree alone; a change anywhere else under `_lym` stays dirty.

### D6 — Only the owner machine writes maps; the stamp stays per environment

`lym gate testmap` refuses on a machine that reads the worktree's entry as owned by another machine, read through `hubgeom`'s `ParentOf` as for a run.
The refusal names the owner machine and the way forward: run it there.
`testcover.EnvironmentStamp` keeps its fields as they are; it holds no machine name or path, so two machines with the same toolchain and hub config read each other's maps as current.
A non-owner machine whose stamp differs reads every map as a `testcover.MissEnvironment` and runs whole packages, and never rewrites the maps.
An owner machine whose stamp changes, as after a Go upgrade, rebuilds and commits every map in its next passes.
Bound: a hub with no owner recorded for the entry reads as owned by no other machine, so two machines can both commit its maps and meet a conflict on the weft pull; recording the owner with `lym adopt` closes it.

### D7 — A pass writes a map only when its content changes

A map whose plan is `UpkeepCurrent` with no dropped entry, no queued test and no cleared failure keeps its file as it is, its `commit` included.
The selector already diffs from the map's own commit and file hashes, so the next pass plans the map from that older commit against the newer HEAD.
Every other map is written as today, its `commit` and format commit moving together.
A map's `commit` now means the commit its content was last written at; the exit record's `planned_head` says which HEAD every map was last planned at.
Rationale: today a pass moves every current map to HEAD, so each landing would commit the whole set; with D7 a landing commits only the maps it changed.
Bound: the commit a map stays at must stay reachable; the warp history a branch carries keeps it reachable.

### D8 — The map file is compact JSON

The map file is written with no indentation, its keys sorted as today.
The reader accepts the indented form too, so the format version stays 2.
Rationale: the indentation puts one integer per line in each block list; compaction cuts the checkout of every weft worktree by about a quarter.

### D9 — `lym gate testmap` runs in the prime or a task worktree

The verb runs in any worktree of a hub that has a weft, on that worktree's own maps at its own HEAD.
It refuses outside a hub, as today, naming the way forward.
The prime's dobby starts the pass automatically in the prime only, as today; in a task, the operator or an agent runs it by hand.
A hand rebuild in a task, such as the one the redundancy sweep runs before judging a package, commits to that task's weft only and never reaches the parent.

### D10 — `test_map_upkeep` is off by default

`internal/dobbyengine/template.yaml` sets `test_map_upkeep: false`.
The operator turns it on per hub with `lym config`.
A hub whose base file holds the key keeps its value; a hub that never set it reads the new default, and its dobby stops starting passes.
Rationale: the operator did not authorise the first 44-minute build, and a build that commits to git is the operator's call.
This is a default change, not a change of the key's meaning, so no `configreg.Reinterpretation` ships.

### D11 — One import of the existing maps into the prime's weft

A call in the prime whose `_lym/testcover/` holds no map, while `<bolt>/.lym/testcover/` holds maps, first reads every `map.json` there and writes it into the prime's directory in D8's compact form, then plans as usual and commits per D4.
Once the prime's maps are committed, any call in the prime removes every `map.json` and its locks from the bolt's old directory while one is left there.
It keeps the old `testmap-exit.json`, which `testmap-upkeep-cost-and-guide` reads as a pass record.
Bound: it runs in the prime only, and imports only while the prime's directory holds no map; a second call imports nothing.
Rationale: it saves a 44-minute rebuild of maps that are current at `79f0eb4a`.

## Constraints

- `PATTERN-lym-files`: the tracked maps go under `_lym`, the never-tracked exit record and locks under `.lym`, at the mirrored subpath; only `internal/lymdirs` names either.
- `PATTERN-fabric-boundary`: the weft commit and push go through `internal/fabricengine` with a positive-only pathspec; D4's verb writes into the weft path fabric hands it and commits at its own end, never through the junction.
- `PATTERN-pair-remote`: a task pair's records push carries its code branch; the prime's stays records-only.
- `PATTERN-owner-machine`: see PATTERN changes.
- `PATTERN-tolerant-records`: the map reader keeps accepting unknown keys; D8 changes no key.
- `PATTERN-refusal-way-forward`: D6's and D9's refusals name the way forward.
- Fact (code at `79f0eb4a`): a pair's weft branch forks from its parent's weft branch and drops only shed run records (`internal/fabricengine/forkrecords.go`); weft branches never merge, and teardown archives a branch's tip with a tag.
- Fact (code at `79f0eb4a`): `testcover.PlanUpkeep` sets `Changed` whenever the map's commit differs from the new commit, so a pass rewrites every current map today.
- Fact (code at `79f0eb4a`): the selector misses with `MissUnmarked` when a map's format commit differs from its commit, and compares the map's recorded file hashes against the target's (`internal/testcover/testselect.go`).
- Fact (code at `79f0eb4a`): the hub env digest hashes the hub's `env.yaml` entries, and the tiers digest the mapped tiers' commands and env (`internal/gatecli/testmappackage.go`); neither holds a machine name.

## Auto-mode assumptions

- A task does not need its maps carried to its own HEAD for its gate; the selector reads the inherited maps against the task's tree.
- The prime's dobby stays the only automatic starter; a beam parent's maps are kept by hand as a task's are.

## Open risks

- Decision for the parent: the board entry's Idea names one extra branch and one extra junction in the weft repository; D1 puts the maps on the existing weft branch instead, because D3 delivers no map by merge. Confirm, or record the extra branch with `lym loom decision add`.
- Decision for the parent: the entry's Idea says raddle splits the work into several tasks because of the new branches; with D1 there is no new branch, and the work may fit one task.
- `testmap-upkeep-cost-and-guide` reads the maps' place, the upkeep default and "which maps are at HEAD"; D1, D7 and D10 change all three, so its guide is right only if it runs after this task lands or follows this record.
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
- D8: a written map is compact JSON, and an indented map is still read.
- D9: `lym gate testmap` in a task worktree plans and writes that task's maps at the task's HEAD.
- D10: a hub with no `test_map_upkeep` key gets no automatic pass from its dobby; a hub that sets it true gets one as today.
- D11: a prime call with an empty `_lym/testcover/` and a populated `<bolt>/.lym/testcover/` imports the maps compact, commits them, and removes the old maps and locks but not the old exit record; a second call imports nothing.
- D11: a prime call whose maps are committed removes old maps still left in the bolt's directory.
- Review: `pattern/PATTERN-test-speed.md` and the `internal/testcover` and `internal/gatecli` package docs describe the maps per worktree in the weft, not in the bolt's `.lym`.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, committing coverage maps, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.

## Notes for the plan writer

- `testcover.Dir` and the callers that pass `fabricengine.BoltDir` are where D1 turns; `internal/gatecli/testmap.go` holds the prime-only refusal D9 widens.
