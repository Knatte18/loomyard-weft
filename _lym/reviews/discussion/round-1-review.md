---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:83
    summary: D9 opens lym gate testmap to task worktrees but leaves a task's pass registered under the hub-wide runproc.HubOwner slot, so it blocks the prime dobby's pass and survives its pair's teardown, against PATTERN-child-processes
    fix: In D9, decide and state how a task worktree's pass registers in internal/runproc. Register it under the task's slug, so pairteardown's runproc.End ends it before the worktree goes, per PATTERN-child-processes. Make the one-pass-at-a-time refusal and the dobby's TestMapRunning read count only passes of the same worktree, so a task's pass neither blocks nor is refused by the prime's. State this as D9's bound. Add an acceptance criterion that a pass in a task does not block the prime's dobby pass and is ended by its pair's teardown.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:98
    summary: D11 imports only while the prime's directory holds no map, so a killed import leaves a partial set that the next call commits, after which the removal deletes the old maps that were never imported
    fix: In D11, replace the guard "imports only while the prime's directory holds no map" with "imports each old map.json whose package has no map in the prime's directory". Remove an old map only once the prime holds a committed map for its package. Restate the Bound that way, and add an acceptance criterion that a call after an import killed partway imports the rest and removes none of them before they are committed.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:63
    summary: D6's claim that two machines with the same toolchain and hub config read each other's maps as current is false in two cases. The env digest stacks env.yaml's machine layer, and a map's commit can be absent from a non-owner's store.
    fix: In D6, replace "it holds no machine name or path, so two machines with the same toolchain and hub config read each other's maps as current" with the two conditions that make sharing hold. First, the HubEnvDigest hashes envconfig's stacked vars, the env.machine.yaml machine layer included (internal/envconfig/config.go, vars is CategoryMachine), so a machine-layer variable makes every map a MissEnvironment on the other machine. Second, a non-owner whose code store lacks a map's commit, such as an unpushed prime HEAD, reads MissCommitUnreadable and runs the package whole. State that both cases degrade to whole-package runs and never to a wrong selection. Correct the matching Constraints fact so it says the hub env digest includes the machine layer.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:68
    summary: D7 leaves an unchanged map at its older commit, but the audit drops every map whose commit is not HEAD (internal/gatecli/testmapaudit.go), so under D7 the audit silently skips every unchanged package
    fix: In D7, add that every reader that compares a map's commit with HEAD to mean the map is current reads the exit record's planned_head instead. Name the audit's candidate filter as one such reader. Add an acceptance criterion that a pass leaving a map unwritten still lets the audit sample that package's tests.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:126
    summary: The board entry's Idea names an extra branch and junction in the weft repository and a raddle split into several tasks. D1 departs from both, and only the parent can confirm the departure.
    fix: Raise to the run's parent both Open-risks lines. First, confirm D1's use of the existing weft branch instead of the board Idea's one extra branch and junction, or record the extra branch with lym loom decision add. Second, say whether raddle still splits the Sketch or the work stays one task. Once the parent answers, drop each answered line from Open risks.
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:75
    summary: D7's Bound says warp history keeps a map's commit reachable but cites no evidence and does not say what happens when an operator rewrite of the prime's code branch drops it
    fix: Rewrite D7's Bound to rest on the evidence. Cite the internal/fabricengine package doc, which says a code branch is never rebased and a rewrite of the prime's branch is reset as a rewrite. Add that a map whose commit is gone from the store reads testcover.MissCommitUnreadable, which an upkeep pass plans as UpkeepMiss and rebuilds whole at HEAD, so the bound degrades to one rebuild and never to a wrong selection.
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:70
    summary: D7's skip condition omits the entry PlanUpkeep adds without queueing, for a new test the spawn scan marks out of process or unresolved, so a literal reading never writes that entry
    fix: In D7, replace "with no dropped entry, no queued test and no cleared failure" with "whose entries, queue and failure record are unchanged by the plan".
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:79
    summary: D8's compact form needs a change to internal/state, whose WriteJSON always indents and through which PATTERN-tolerant-records routes the map write. Its keys sorted as today is inaccurate, since struct fields marshal in declaration order.
    fix: In D8, state that internal/state's JSON writer gains a compact form the map writer uses, keeping the unknown-key carry-over PATTERN-tolerant-records requires. Replace "its keys sorted as today" with "its field order as today".
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:50
    summary: D4 puts the map locks under the worktree's .lym/testcover/, but lock.TrackedFileLockPath puts a _lym file's lock under the checkout's .lym/locks/ mirror. The exit record, which shares testcover.Dir with the maps today, needs its own .lym accessor, which D1 and D4 never name.
    fix: In D4, replace "The exit record, the locks and the build locks stay never-tracked, under the worktree's .lym/testcover/" with this. The exit record and the build locks stay never-tracked under the worktree's .lym/testcover/, through a scratch accessor beside testcover.Dir. Each map's lock is the never-tracked one lock.TrackedFileLockPath gives.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:100
    summary: With D10's default off and this hub's beam.yaml holding no test_map_upkeep, no automatic call makes D11's first prime call, so every gate runs whole packages from the landing until someone runs the verb by hand
    fix: In D11, name who makes the first prime call after the landing, such as the operator by hand with lym gate testmap in the prime. Alternatively, make D11's import run from the first call of any map reader in the prime. State the window between the landing and that call in Open risks.
  - id: F11
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:61
    summary: The record never orders D6's owner refusal against D11's import and removal. Import first would write maps on a non-owner. Refusal first never removes a non-owner's old bolt maps.
    fix: In D11, state that the owner refusal of D6 comes before the import and the removal. Also state what becomes of a non-owner machine's own bolt .lym/testcover/ maps, either left in place or removed by any prime call on that machine before the refusal.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:93
    summary: D10 says per hub and names only a hub's base file. beam.yaml is a per-worktree module, and test_map_upkeep is open to the prime's worktree file and its local layer too.
    fix: In D10, replace "The operator turns it on per hub with lym config. A hub whose base file holds the key keeps its value; a hub that never set it reads the new default" with this. The operator turns it on with lym config in the prime. A layer that holds the key keeps its value, and a prime whose layers never set it reads the new default.
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:71
    summary: D7 restates two facts Constraints already holds, that the selector diffs from the map's commit and file hashes and that a pass today rewrites every current map
    fix: In D7, cut the sentence "The selector already diffs from the map's own commit and file hashes, so the next pass plans the map from that older commit against the newer HEAD." to "The next pass plans the map from that older commit against the newer HEAD." Cut the rationale's "today a pass moves every current map to HEAD, so each landing would commit the whole set;" so the rationale reads only "with D7 a landing commits only the maps it changed."
---

### [BLOCKING] D9 leaves a task's pass on the hub-wide registration

**Location:** `_lym/discussion/decision-record.md:83` (D9)

**Issue:** `registerUpkeepPass` (`internal/gatecli/testmap.go:318-352`) registers every upkeep pass as `runproc.RoleTestMap` under `runproc.HubOwner`, and it refuses while any other process holds that registration.
The prime dobby's `TestMapRunning` reads the same registration (`internal/beamcli/dobbydeps.go:248`, `internal/dobbyengine/testmap.go:54-60`).
Once D9 opens the verb to task worktrees, a hand pass in a task can run for up to 44 minutes, and during that time it blocks the prime's upkeep.
A prime pass also refuses every task's pass, with a message that names the hub.
Registered under `HubOwner` rather than the task's slug, the pass is never ended by `pairteardown`'s `runproc.End(slug)`, so it keeps writing into a worktree being removed.
`PATTERN-child-processes` requires that a run's processes register in `internal/runproc` and end only through it.
D9 is silent, so the existing hub-wide registration carries over and breaches that constraint.
Three lenses found this independently.

### [MEDIUM] D11's no-map guard loses the rest of a killed import

**Location:** `_lym/discussion/decision-record.md:98` (D11)

**Issue:** D11 imports only while the prime's `_lym/testcover/` holds no map.
A call killed partway through leaves a partial set, and D5 keeps that set from making the worktree dirty.
The next call therefore imports nothing and commits the partial set per D4.
D11's removal then deletes every old `map.json` still in `<bolt>/.lym/testcover/`.
The packages that were never imported read `MissNoMap` and need the rebuild D11 exists to avoid.
The Bound states no rule for this case, so the D5/D11 focus question settles against the record.

### [MEDIUM] D6's machine-sharing premise ignores the machine env layer and commit availability

**Location:** `_lym/discussion/decision-record.md:63` (D6) and the Constraints fact at line 117

**Issue:** `stampEnvironment` hashes the entries of `envconfig.LoadConfig(fabricengine.ConfigDirs(...))` (`internal/gatecli/testmap.go:362`, `internal/gatecli/testmappackage.go:818`).
env.yaml's `vars` key is `CategoryMachine` (`internal/envconfig/config.go:65`), so `env.machine.yaml` is stacked into `HubEnvDigest`.
`PATTERN-config-layers` gives a path on the machine's disk as the typical machine-layer value.
The fact "neither holds a machine name" and D6's "two machines … read each other's maps as current" are therefore false on a hub with machine-layer vars.
Separately, `commitMiss` (`internal/testcover/testselect.go:260-273`) misses on a map whose commit the reader's store lacks.
The prime's code push is the operator's, so a non-owner can lack the commit the owner mapped at.
Both cases degrade to whole-package runs, which the record should state rather than claim sharing outright.

### [MEDIUM] D7's new meaning of `commit` breaks the audit's candidate filter

**Location:** `_lym/discussion/decision-record.md:68` (D7)

**Issue:** The audit keeps only maps whose `Commit` equals HEAD: `m.Commit != r.head` skips the map (`internal/gatecli/testmapaudit.go:38`).
D7 deliberately leaves current maps at their older commit and moves the "planned at HEAD" meaning to the exit record's `planned_head`.
Every package a pass leaves unwritten therefore drops out of the audit, and the record never says so.
D7's acceptance criterion checks only the planner.

### [MEDIUM] The extra-branch departure and the raddle split need the parent

**Location:** `_lym/discussion/decision-record.md:126` (Open risks)

**Issue:** The board entry's Idea names "one extra branch and one extra junction" and says raddle splits the work into several tasks because of the new branches.
D1 drops both on the strength of the operator's Direction, under which maps are not delivered.
The reasoning holds, but it departs from an explicit operator statement on the entry, so only the parent can confirm it.
Both Open-risks lines are real parent questions, not ones the writer could settle, and they go to the parent together because the split question follows from the branch question.

### [LOW] D7's reachability bound cites no evidence and no failure outcome

**Location:** `_lym/discussion/decision-record.md:75` (D7 Bound)

**Issue:** The `internal/fabricengine` package doc states that a code branch is never rebased.
It also states that an upstream rewrite of the prime's branch is detected and reset, and that a rebased-away object survives a fetch.
The object is then reachable only through reflogs until a teardown's `git gc --auto` prunes it.
After that, `commitMiss` gives `MissCommitUnreadable`, and `PlanUpkeep` returns `UpkeepMiss`, which rebuilds the package whole (`internal/testcover/upkeep.go:22,74`).
The bound holds in substance, but the record asserts it without that evidence or that outcome.
The D7 focus question therefore settles as "claim true, evidence missing".

### [LOW] D7's skip condition is narrower than PlanUpkeep's changes

**Location:** `_lym/discussion/decision-record.md:70` (D7)

**Issue:** `PlanUpkeep` adds an entry without queueing it for a newly declared test that the spawn scan marks out of process or unresolved (`internal/testcover/upkeep.go:114-117`).
D7's list of changes that force a write does not name that case.

### [LOW] D8's compact write needs `internal/state`

**Location:** `_lym/discussion/decision-record.md:79` (D8)

**Issue:** `testcover.WriteMap` goes through `state.WriteJSON`, whose `writeJSONUnlocked` always calls `json.Indent` (`internal/state/state.go:94-99`).
`PATTERN-tolerant-records` routes the rewrite through `internal/state`, so D8 rests on a change to a shared writer that it does not name.
The reader side holds: `ReadJSON` ignores whitespace, and `mapMiss` checks only `Format.Version`, so version 2 stays valid.
"Keys sorted as today" is inaccurate, because struct fields marshal in declaration order.

### [LOW] D4 misplaces the map locks and leaves the exit record's accessor unnamed

**Location:** `_lym/discussion/decision-record.md:50` (D4)

**Issue:** `lock.TrackedFileLockPath` (`internal/lock/tracked.go:23-40`) puts the lock of a file under `_lym` in the checkout's `.lym/locks/` mirror, not in `.lym/testcover/`.
Today `testcover.Dir` serves both the maps and the exit record.
D1 moves `Dir` to the tracked side, while D4 keeps the exit record on the untracked side, so a second accessor is needed, and neither decision names it.

### [LOW] Nothing makes D11's first prime call once D10 turns upkeep off

**Location:** `_lym/discussion/decision-record.md:100` (D11)

**Issue:** D1 stops every reader from reading the bolt's maps, and D10 stops the dobby's automatic pass.
This hub's `_bolt/_lym/config/beam.yaml` holds no `test_map_upkeep`.
Until someone runs `lym gate testmap` in the prime by hand, every gate reads `MissNoMap` and every new pair inherits no maps.
The record names no one who makes that call.

### [LOW] D6's refusal and D11's import are unordered

**Location:** `_lym/discussion/decision-record.md:61` (D6, D11)

**Issue:** If the import runs before the refusal, a non-owner machine imports its own bolt maps and writes maps, which D6 forbids.
If the refusal comes first, a non-owner's old bolt directory is never removed.
The record states neither order.

### [LOW] D10's layer wording

**Location:** `_lym/discussion/decision-record.md:93` (D10)

**Issue:** `beam.yaml` is a per-worktree module with `test_map_upkeep` classed `CategoryNone` (`internal/dobbyengine/config.go:64-77`), so the prime's worktree file and its local layer can also hold the key.
The default flip itself is backed: `PATTERN-config-layers` says a key the stacked layers lack resolves to its template default.
That settles the D10 focus question in the record's favour except for this wording.

### [NIT] D7 restates Constraints facts

**Location:** `_lym/discussion/decision-record.md:71` (D7)

**Issue:** D7's sentence on how the selector diffs, and the first half of its rationale, restate the second and third Constraints facts.
The plan writer reads Constraints, so the cut loses nothing (the completeness-before-leanness test holds).

## Focus verdicts not raised as findings

- D5: the exemption states its bound, the `_lym/testcover/` subtree alone, and `preflight.runStateExemptRels` is the one place that decides it. It holds after a killed call, since D4 commits the leftover maps on the next call.
- D4/D6: only `internal/gatecli/testmappackage.go` writes maps, and both of its write sites run after the location check where the refusal goes. The verb's own commit is scoped to `_lym/testcover/` through `fabricengine.CommitRecordsPaths`. The other `_lym` records commits that sweep in maps are already named.
- Open risks figures: the 41 MB, 0.8 GB and 44-minute figures are measurements backing a stated cost or rationale, not perishable pins a reader must keep current.

## Rejected

- existing-paths noted that a layer still holding a retired non-zero `test_map_every` reads as off after D10. `MigrateConfig` drops that key, and off is D10's intended default, so this is no defect.
- bound-breaker's second D6 finding is merged into F3, not kept separately: it attacks the same sharing claim.
