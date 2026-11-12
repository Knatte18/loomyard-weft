---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:50
    summary: D4 puts the map build locks under a scratch accessor beside testcover.Dir, against PATTERN-lym-files' rule that a tracked file's lock path comes from lock.TrackedFileLockPath under the checkout's .lym/locks/ mirror
    fix: In D4 (decision-record.md:50), change the sentence to say that only the exit record stays never-tracked under the worktree's .lym/testcover/ through a scratch accessor beside testcover.Dir, and that each map's build lock, like its write lock, is the one lock.TrackedFileLockPath gives with the build suffix, under the checkout's .lym/locks/ mirror. In the Constraints PATTERN-lym-files bullet (line 142), replace "the never-tracked exit record and build locks under .lym, at the mirrored subpath, and each map's lock under the checkout's .lym/locks/" with "the never-tracked exit record under .lym at the mirrored subpath, and each map's write and build locks under the checkout's .lym/locks/ through lock.TrackedFileLockPath".
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:101-105
    summary: D9 says a task's pass registers under the task's slug so that teardown ends it, but today only a --upkeep call registers; the hand calls D9 describes in a task, the sweep's named rebuild included, register nothing and outlive the teardown
    fix: "In D9, replace the sentence \"A pass registers in internal/runproc as runproc.RoleTestMap under its worktree's owner...\" with a rule that covers every call in a task: every lym gate testmap call in a task worktree, a named-package rebuild included, registers itself as runproc.RoleTestMap under the task's slug, and only a --upkeep call does the one-pass-at-a-time check. Keep the prime as today: a --upkeep call registers under runproc.HubOwner. State the bound: a call in a task that the teardown ends leaves its maps uncommitted for the teardown's pending-records commit. Reword the D9 acceptance criterion at line 181 to: a hand lym gate testmap call in a task, with and without packages named, is ended by its pair's teardown, and a pass in a task does not block the prime's dobby pass."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:28,161-162
    summary: The two arbiter Added-after-Discussion entries settle both Decision-for-the-parent Open-risks lines and order them dropped and D1's rationale to cite the decision, but both lines remain and D1's rationale does not cite it
    fix: "Delete the Open-risks bullet that begins \"Decision for the parent: the board entry's Idea names one extra branch\" and the one that begins \"Decision for the parent: the entry's Idea says raddle splits the work\". In D1's rationale (line 28), add that the arbiter's Added-after-Discussion decision \"D1 stands\" (2026-11-12) confirms the existing weft branch with no extra branch or junction. Leave both Added-after-Discussion entries untouched."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:47
    summary: D4 has a hand lym gate testmap call in a task pair spawn a push of the pair's code branch, with no run-scope origin reconcile and no stated bound, though PATTERN-owner-machine gates a loom pair's push on that reconcile
    fix: "In D4 (line 47), choose and state one of: (a) the verb's push in a task pair is records-only, and the code branch goes with the run's next records push; or (b) the push in a task pair first runs the run-scope origin reconcile PATTERN-owner-machine names, and spawns the code-branch push only when the reconcile reads the entry as active. With (a), say how this squares with PATTERN-pair-remote's rule that a pair's records push carries its code branch, and update the Constraints PATTERN-pair-remote bullet to match. With (b), add an acceptance criterion: a testmap call in a task whose entry origin reads as ended pushes no code branch."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:48-49,57
    summary: Uncommitted maps in the prime's weft, while a pass runs and after a killed call, are the dirt fabric's weft pull stops on, through weftDirtyStop and the replay's ErrReplayDirty; the record calls the leftover harmless and is silent on this path
    fix: In D5 (or D4), add one decision sentence for how a weft pull meets uncommitted _lym/testcover/ paths. Either a records commit of _lym/testcover/ runs before the weft move, under fabric's records write lock, as D4's own commit does; or the stop stands, and the weft dirty stop names lym gate testmap, whose end commit clears it, as its way forward. State the bound; the subtree is the only one either route covers. Add an acceptance criterion for the chosen behaviour, such as a prime sync whose weft has only uncommitted _lym/testcover/ changes and an origin that moved.
  - id: F6
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:79-82
    summary: D7 has an audit job write the planned map at HEAD, but the package job reads the on-disk map, which under D7 still holds the older commit's Files, OtherFiles and Linked; retargeting the guard alone writes old file hashes under the new commit
    fix: In D7, after the sentence about the audit writing the planned map at HEAD, add that a package job measuring tests into a map the pass left unwritten starts from the plan's map at the pass's HEAD (PlanUpkeep's result, with Commit, Files, OtherFiles and Linked moved to the tree), never from the file on disk.
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:77-81
    summary: Under D7 an unwritten current map keeps its older BuiltAt and Files, so the redundancy step's MapAge and mapDrift report age and drift that the last plan judged irrelevant; D7 is silent on these readers
    fix: In D7, add one sentence that the redundancy step's map age and drift, which internal/gatecli/redundancy.go reads, describe the map's last write, not whether it is current, and state whether the step keeps them as they are or reports the exit record's planned_head beside them.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:117-118
    summary: D11's import checks that a package has no map without taking the package's build lock, so a hand call can overwrite the map a live pass just wrote with the old bolt map
    fix: In D11, add that the import takes each package's build lock without waiting, checks again that the package has no map in _lym/testcover/, and skips a package whose lock another call holds. Add to its Bound that a skipped package is imported by a later call.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:25-26,45
    summary: D1 says testcover.Dir takes the worktree's anchor and every reader and writer, the upkeep pass included, works through it, while D4 says the verb writes into the weft worktree's own _lym path and never through the anchor's junction
    fix: In D1 (line 25), say that the readers resolve the maps through the worktree's anchor, and that the writer, lym gate testmap, writes into the weft worktree's _lym path that fabricengine hands it, as D4 states.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:126
    summary: D11 calls the bolt maps current at 79f0eb4a, but they record commit 5e41ee2b, a descendant with no Go diff
    fix: In D11's rationale (decision-record.md:126) and the support log's Rejected alternatives line about rebuilding (support-log.md:77), replace "current at 79f0eb4a" with "built at 5e41ee2b, which has no Go change over 79f0eb4a".
---

### [BLOCKING] D4 places the build locks outside `lock`

**Location:** `_lym/discussion/decision-record.md:50`, and the Constraints `PATTERN-lym-files` bullet at line 142.

**Issue:** `pattern/PATTERN-lym-files.md` (Tracked-file locks) says a `_lym` file's lock path comes from `lock.TrackedFileLockPath` under the checkout's `.lym/locks/`, mirroring the file's path.
Under D1 the map is a tracked `_lym` file.
Today the build lock is that kind of lock: `testMapBuildLockPath` returns `lock.TrackedFileLockPath(testcover.MapPath(mapsDir, pkg), ".build.lock")` (`internal/gatecli/testmap.go:674-676`).
D4 instead puts the build locks under `.lym/testcover/` through a testcover scratch accessor, while the next line routes "each map's lock" through `lock`.
One map's two locks would then follow two path schemes, and the build lock would not come from `lock`.
The `PATTERN-lym-files` constraint makes this BLOCKING.
The wording came from the fix round 1's F9 proposed, which the fixer applied as written.
That proposal kept the build locks beside the exit record, and it is wrong for the same reason: the build lock locks a tracked `_lym` file.

### [MEDIUM] D9's registration does not cover the hand calls it describes

**Location:** `_lym/discussion/decision-record.md:101-105` and the D9 acceptance criterion at line 181.

**Issue:** `runTestMap` calls `registerUpkeepPass` only when `request.upkeep` is set (`internal/gatecli/testmap.go:252-256`).
`--upkeep` is hidden (`testmap.go:134-137`) and refuses named packages (`testmap.go:238-239`).
D9 says that in a task "the operator or an agent runs it by hand", and it cites the redundancy sweep's named rebuild.
None of those calls registers, so `endOwnedProcesses`, which reaches only registered records (`internal/pairteardown/teardown.go:697-698`), never ends them.
`RemovePair` then deletes the worktree under a live call, whose `lockPackage` `MkdirAll` (`testmap.go:681`), map writes and D4's end commit run against the removed paths.
D9's `PATTERN-child-processes` claim therefore holds only for a `--upkeep` call, and the record never says a task call uses that flag.
Three lenses found this independently.
This is a gap in round 1's F1 fix, which applied registration under the slug to passes but not to the unregistered hand calls.

### [MEDIUM] Arbiter-settled Open-risks lines are still in the record

**Location:** `_lym/discussion/decision-record.md:161-162` and D1's rationale at line 28.

**Issue:** The `### Added after Discussion` entries at lines 128-138 settle both questions and tell the fixer to drop those two lines and to cite "D1 stands" in D1's rationale.
Both lines remain, and D1's rationale cites only the operator's direction.
The rubric's Finding class section makes an Open-risks line that such an entry settles a `consistency` finding, fixed by dropping the line.

### [MEDIUM] D4's code-branch push in a task pair has no stated gate or bound

**Location:** `_lym/discussion/decision-record.md:47`.

**Issue:** D4 has the verb spawn the pair's detached records push "with the code branch in a task pair".
`pattern/PATTERN-owner-machine.md:14-20` makes a loom run's commit and push of its pair proceed only after a run-scope origin reconcile reads its entry as active.
D6 checks only the owner machine through `ParentOf`.
A hand call in a task whose entry origin has ended or paused would still push the code branch.
`PushWarpRebaseFreeAt` skipping a branch origin lacks or has rewound (`internal/fabricengine/spawn.go:225-228`) is the only limit, and the record states none.
D4 adds a push edge reached from a new place, the task worktree that D9 opens, so the rubric's attack-surface rule needs its bound.
I judged it MEDIUM, not BLOCKING. The reconcile rule binds a loom run's push, and whether a hand verb's push in a pair counts as one is a design call the record has to make, not a plain violation.

### [MEDIUM] Uncommitted maps stop the prime's weft pull

**Location:** `_lym/discussion/decision-record.md:48-49` and D5's rationale at line 57.

**Issue:** D5 exempts `_lym/testcover/` from preflight's clean check only.
Fabric's weft pull has two dirty stops:
- when the pull fails and the weft has dirty paths, it stops through `weftDirtyStop`;
- when both sides moved, the replay fails with `ErrReplayDirty`, which also ends in `weftDirtyStop` (`internal/fabricengine/pull.go:406-415`, `:440-445`).

A pass in the prime leaves its rewritten maps uncommitted for as long as it runs, which can be tens of minutes, and a killed call leaves them until the next call.
The record calls that leftover harmless and does not say how a prime sync meets it.

### [LOW] D7's audit job starts from the on-disk map

**Location:** `_lym/discussion/decision-record.md:79-82`.

**Issue:** The non-whole package job reads the map file (`internal/gatecli/testmappackage.go:81-87`).
Under D7 that file keeps the older commit's `Files`, `OtherFiles` and `Linked`.
Only `PlanUpkeep`'s in-memory result moves those fields to the tree (`internal/testcover/upkeep.go:79-83`).
D7 retargets the guard to the planned HEAD but never says the job starts from the plan's map.
Taken literally, the job would write the old hashes under the new commit.

### [LOW] Redundancy step's age and drift misread an unwritten map

**Location:** `_lym/discussion/decision-record.md:77-81`.

**Issue:** `mappedPackages` reports `MapAge` from `BuiltAt` and `Drift` from the map's `Files` against the merge base (`internal/gatecli/redundancy.go:315-318`, `:346-364`).
Under D7 a current map keeps both from its last write, so the report shows staleness the last plan ruled out.
D7 names only the readers that compare `commit` with HEAD.

### [LOW] D11's import check runs without the build lock

**Location:** `_lym/discussion/decision-record.md:117-118`.

**Issue:** A hand prime call can find no map for a package while a live pass holds that package's build lock (`internal/gatecli/testmap.go:674-685`) and then writes its new map.
The import then writes the old bolt map over it.
The cost is rework, since the next plan diffs from the old commit, and never a wrong selection.
The record states neither the lock nor this bound.

### [LOW] D1 and D4 name different path sources for the writer

**Location:** `_lym/discussion/decision-record.md:25-26`, `:45`.

**Issue:** D1 has every reader and writer, the upkeep pass included, go through `testcover.Dir` on the worktree's anchor, whose `_lym` is the junction.
D4 has the verb write into fabric's weft path and never through the junction.

### [NIT] D11 names the wrong commit for the bolt maps

**Location:** `_lym/discussion/decision-record.md:126`, `_lym/discussion/support-log.md:77`.

**Issue:** The bolt maps record commit `5e41ee2b`.
`git merge-base --is-ancestor 79f0eb4a 5e41ee2b` confirms it descends from `79f0eb4a`, and the lens found no Go diff between them, so D11's premise holds and only the stated commit is wrong.

## Focus items

- D9 and runproc: F2. The dobby's `TestMapRunning` lists `runproc.HubOwner` only (`internal/beamcli/dobbydeps.go:619-625`), and the registration lock sits in the hub's scratch directory. A task's pass registered under its slug therefore neither blocks nor is refused by the prime's, as D9's bound says.
- D11 and a killed import: no loss found.
  An imported map left uncommitted stays on disk.
  The next call does not import it again, since the package now has a map, and D4's commit takes it.
  The old map is removed only once a committed counterpart exists.
  The weft-pull stop that the leftover can cause is F5.
- D6: the record states both the hub env digest over envconfig's stacked `vars` (`internal/envconfig/config.go`, machine category) and `MissCommitUnreadable` for an absent commit (`internal/testcover/testselect.go` `commitMiss`), and says both degrade to whole-package runs. Held.
- D7: the audit filter (`internal/gatecli/testmapaudit.go:38`) and the package guard (`internal/gatecli/testmappackage.go:82`) are the readers that compare commit with HEAD, and the D7 audit criterion reaches both. `FailedAt` compares too, but only on a failed build, which is always written. The gaps are F6 and F7.
- Order:
  - D11's removal runs on any machine before D6's refusal, and the import runs after it, so a non-owner never writes a map.
  - D11 has the operator make the first prime call by hand, since D10 turns the automatic pass off, and Open risks states the window until then.
  - A non-owner's own bolt maps are removed once its pulled weft holds committed counterparts.
  - This machine, `neve`, is the entry's owner and holds the bolt maps, so the import has its source.
  - Held.

## Rejected

- No other finding was rejected. Duplicate reports of the D9 registration gap from the existing-paths, walkthrough and bound-breaker lenses are merged into F2.
