---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 second bullet, with D1 base per tier and D2 staleness
    summary: Map freshness is checked against the tier's base while maps are only built at prime main HEAD, so selection misses in the flows the board names
    fix: >-
      Rework D4 and D5 so selection and freshness use the map's own recorded `Commit` instead of the tier's base.
      For each importing package, D4 finds the changed functions of each covered package between the map's `Commit` and the working tree,
      reading content through `fabricengine.Fabric.CodeFileAt`; D5 drops the hash check against the tier's base
      and widens only when the map's `Commit` is not readable in the worktree's object store.
      State in D5 that a change on main since the map was built counts as a change, which can only add tests.
      Rewrite the D5 acceptance criteria to match, and drop the Open risks hit-rate line, which the change answers.
      If the writer keeps the tier-base check instead, record the measured hit rate as a Constraint with its evidence:
      52 of 163 main commits from 2026-11-04 to 2026-11-11 touched configreg, loomengine, gateslot, fabricengine or state.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 second bullet and D2 second bullet
    summary: D5 can be read to skip the importing package's own Files, so a test edited on main since the map build keeps stale blocks and is not selected
    fix: >-
      In D5's second bullet, state that the importing package's own recorded `Files`, test files included, is compared as well as every covered package's field,
      and that any difference is a miss for that package.
      Add that case to the D5 acceptance criterion.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1 Bound and D3 third bullet
    summary: D3 relies on the spawn scan to find out-of-process tests, but the record never states the scan's blind spots, which let tests through unselected
    fix: >-
      Add a Bound to D3 naming what the spawn scan does not see: a call into another in-module package that is not a testkit is not followed,
      every test in a tier0 file gets an empty verdict, and a hook or alias that makes git run module code is not seen.
      A test that runs changed code in such a child process is not selected, and only the landing gate catches it.
      Cite `testcover.ScanSpawns` as the source of the verdict in D2's third bullet.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2 first and fourth bullets
    summary: testmap skips dirty files and checks files changed under tests only in the package's own directory, so a covered package's hashes can describe code other than what was built
    fix: >-
      In D2, state that testmap skips a package when any covered package holds a modified or untracked Go file,
      and that the check for files changed under the tests covers every covered package's files.
      Alternatively, D2 records each covered package's hashes from the files the build read.
      Add a D2 acceptance criterion for the dirty covered-package case.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2 fourth bullet and Constraints PATTERN-tolerant-records bullet
    summary: WriteMap keeps undeclared keys, so an older binary's rewrite keeps the new format marker over a map without cross-package blocks
    fix: >-
      In D2, bind the format marker to the build that wrote it, for example a marker holding the same `Commit` and `BuiltAt` as the map's top-level fields,
      and state in D5 that a marker that does not match those fields is a miss.
      Add the mismatch case to the D5 acceptance criterion.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Constraints PATTERN-fabric-boundary bullet
    summary: The constraint names the impactset diff as the path that reads base file content, but that path reads only file names
    fix: >-
      Rewrite the bullet to say base file content for D4 is read through `fabricengine.Fabric.CodeFileAt`,
      the reader testmap's file hashes and the redundancy report's map drift already use, never a new git call.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks second bullet, and support-log.md Q8
    summary: The question on this hub's gate.yaml switch is a choice the writer can make, and Scope Out already excludes the hub's gate.yaml
    fix: >-
      Drop the second Open risks bullet.
      Extend the Scope Out bullet on this hub's `gate.yaml` to say that the operator switches the three keys to `coverage` with `lym config gate` from the prime after landing.
      Mark Q8 in the support log's question ledger as an auto-pick with that answer.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks first bullet
    summary: The question for the parent says note testmap-upkeep orders itself before this task, but the note only claims the whole staleness policy
    fix: >-
      Reword the first Open risks bullet: D2 changes testmap's staleness rule and build cost,
      and note `testmap-upkeep` says it owns the whole staleness policy, dependency drift included.
      Keep the question to the parent of whether D2 may change that policy now.
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Auto-mode assumptions first bullet and Open risks last bullet
    summary: The record assumes the cost is in tiers 0 and 1 and leaves the tier 2 share unmeasured, though the gate run records hold it
    fix: >-
      Replace the first Auto-mode assumption and the last Open risks bullet with one Constraint stating the measured split and its source.
      The darn-write tier run records under the hub's gate runs directory from 2026-11-04 on, build, guard and carried records left out, sum to
      tier0 191 min, tier1 516 min and tier2 268 min over about 245 runs each, so tier 2 is about 27 percent.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 fourth bullet
    summary: A named import that is added or removed can add or drop a package and its init from the binary, yet D5 counts named imports as no miss
    fix: >-
      In D5's fourth bullet, count an added or removed named import as a miss, as a blank or dot import is,
      and keep only a reordered or regrouped import block as no miss.
      Update the D5 acceptance criterion that names named imports to match.
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 first paragraph and D5 fifth bullet
    summary: Several init functions in one file share the key name init, so pairing them by name between base and working tree is undefined
    fix: >-
      In D4, state that functions sharing a key name, such as several `init` functions, count as changed together when any of them differs or their count differs.
      In D5's fifth bullet, say a changed file whose count of `init` functions grew is a miss.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, first D3 bullet
    summary: The D3 criterion says an importing package runs nothing outside the union but the same-name overlap, which contradicts D6's over-budget whole run
    fix: >-
      Extend the first D3 criterion to allow D6's over-budget whole run beside the same-name overlap.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7
    summary: D7's labels have none for a package run whole for its lym binary tests or for a selected package run whole past the command budget
    fix: >-
      Add the labels `whole: lym binary` and `whole: over budget` to D7 and to the D7 acceptance criterion.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, first D5 bullet
    summary: The D5 criterion naming a new file in a covered package can be read as a file the diff adds, which D4 and D5 treat as no miss when it holds only plain functions
    fix: >-
      Reword the first D5 criterion to name a Go file added to a covered package between the map's build and the scope's comparison point.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Notes for the plan writer, last bullet
    summary: The hint to reuse testcover.PackageBlocks is wrong, since the judgment D2 changes reads the map's block keys, not profile blocks
    fix: >-
      Rewrite the last Notes bullet: the redundancy judgment builds its kept set from the map's `Blocks` keys,
      and the own-package filter keeps the keys that carry no import path.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Constraints last bullet
    summary: The PATTERN-test-speed and PATTERN-test-isolation bullet restates the PATTERN rules and adds nothing specific to this task
    fix: >-
      Cut the bullet, or keep only a task-specific part, such as the fixture module of the D2 criteria needing hermetic git.
---

### [BLOCKING] Map freshness against the tier's base defeats selection in the board's flows

**Location:** `_lym/discussion/decision-record.md` D5, second bullet, read with D1's base per tier and D2's staleness rule.

**Issue:** Maps are built only in the prime, at main's HEAD (`internal/gatecli/testmap.go`, `testMapLocation` and `mapPackages`), and D2 makes a map stale, and so rebuilt at the newest HEAD, whenever any covered package changes.
D5 widens to a whole package whenever a covered package's files at the tier's base differ from the map.
Two common cases then always miss:

- A worktree whose merge base predates a landing that touched a widely imported package.
  From 2026-11-04 to 2026-11-11, 52 of main's 163 commits touched `internal/configreg`, `internal/loomengine`, `internal/gateslot`, `internal/fabricengine` or `internal/state`, so most maps are rebuilt past a running branch's base within hours.
- A tier base that `newestWholeAncestor` (`internal/gatecli/green.go:202`) takes from a branch commit, such as `webster-fix` after a `webster` whose tiers 0 and 1 passed.
  The branch's own earlier changes to a package make that package's hashes differ from every map built on main.

In both cases the gate runs as `diff` does today, the outcome the board entry files as the bug.
The Open risks hit-rate line names the effect but leaves the fact the design rests on unmeasured, and no Decision reconciles D2 with D5.
The map already records its `Commit` (`internal/testcover/coveragemap.go:27`).
Diffing from that commit keeps "widen, never narrow", because a change on main since the build only adds tests.

### [MEDIUM] The importing package's own files are not plainly part of the freshness check

**Location:** `_lym/discussion/decision-record.md` D5, second bullet, and D2, second bullet.

**Issue:** D2 records the new hashes "per covered package other than its own", and D5 checks "for a covered package".
An implementer can check only the new field.
Suppose a test T of B is edited on main after B's map was built, and now calls `A.G`.
A branch then edits `A.G`.
B is not a changed package, T is still in the map, and its recorded blocks miss `A.G`, so T is not selected.
`Map.Files` holds B's test files (`fileHashes` hashes every `.go` file), so comparing it closes the gap.

### [MEDIUM] The spawn scan's blind spots are an unstated let-through

**Location:** `_lym/discussion/decision-record.md` D1 Bound and D3, third bullet.

**Issue:** D3 runs a test the map marks out-of-process or unresolved, and that verdict comes from `testcover.ScanSpawns`.
`analyzeCall` returns an empty verdict for a call through an import name (`internal/testcover/spawnscan.go`, the `*ast.SelectorExpr` case).
A test of B that calls production code in A, which starts a process running module code, is therefore mapped as in-process.
Its profile misses the code the child ran, so a change there does not select it.
The scan's own doc names a further blind spot: hooks and aliases that make git run module code.
The rubric requires a Decision that lets tests through to state what it lets through and what bounds it.
D1's bound names only importing packages and the landing gate, not this class.

### [MEDIUM] testmap's dirty and changed-under-tests checks cover only the package's own directory

**Location:** `_lym/discussion/decision-record.md` D2.

**Issue:** `dirtyGoFile` (`internal/gatecli/testmap.go`) and `changedFiles` (`internal/gatecli/testmappackage.go`) look only at the package's own directory.
With a cross-package coverpkg, `go test -c` builds and `KeyBlocks` reads the prime's working tree for every covered package, while D2 would record HEAD hashes for them.
A modified file in another package of the prime yields a map whose hashes match HEAD but whose blocks describe different code.
D5 then trusts it.

### [MEDIUM] The format marker survives a rewrite by an older binary

**Location:** `_lym/discussion/decision-record.md` D2, fourth bullet; Constraints, PATTERN-tolerant-records bullet.

**Issue:** `testcover.WriteMap` "keeps each key of the replaced record that Map does not declare" (`internal/testcover/coveragemap.go:66-72`).
An older `lym gate testmap` that rewrites a map with own-package blocks therefore carries the new marker and covered-package field over.
The gate reads the result as a new-shape map with no cross-package blocks, selects nothing for importers and sees no miss.
The marker as designed does not tell such a map apart.

### [MEDIUM] The fabric-boundary constraint names a path that reads no file content

**Location:** `_lym/discussion/decision-record.md` Constraints, PATTERN-fabric-boundary bullet.

**Issue:** impactset reaches git only through `gitexec.Run` with `rev-parse`, `merge-base`, `diff --name-status` and `ls-files` (`internal/impactset/derive.go`, `internal/impactset/targets.go:109,118`).
None of these returns a file's content.
The existing content reader is `fabricengine.Fabric.CodeFileAt` (`internal/fabricengine/codereads.go:27`), which testmap and the redundancy report already use.
As written, the constraint cannot be met without the new git call it forbids.

### [MEDIUM] The gate.yaml question is a writer's choice that Scope already made

**Location:** `_lym/discussion/decision-record.md` Open risks, second bullet; `support-log.md` Q8.

**Issue:** Scope Out already excludes "This hub's own `gate.yaml`", and the run can change no hub config.
Whether the operator flips the keys after landing is a choice the writer can record, not a question only the parent can settle.
No board note titled `Decision for gate-scope-by-coverage-map:` and no `Added after Discussion` entry answers it.
Per the rubric it is still not `decision` class, so the Open risks line goes.

### [LOW] The testmap-upkeep question misstates the note

**Location:** `_lym/discussion/decision-record.md` Open risks, first bullet.

**Issue:** The bullet says note `testmap-upkeep` "orders before this task".
The note says the current upkeep is too large a job and that it "owns the whole policy", item 4 included, which asks whether dependency drift needs more than the age limit.
D2 changes that very rule, so the question is a real conflict with another entry and stays for the parent.
Only its premise is misquoted.
No board note titled `Decision for gate-scope-by-coverage-map:` answers it yet.

### [LOW] The tier 2 share is assumed, though the run records hold it

**Location:** `_lym/discussion/decision-record.md` Auto-mode assumptions, first bullet; Open risks, last bullet.

**Issue:** Whether tiers 0 and 1 carry the gate's cost is a fact the design rests on, and the record leaves it as an assumption plus a risk line.
The gate's `RunRecord`s carry `gate_key`, `tier` and `wall_seconds`.
On this machine, darn-write tier runs from 2026-11-04 on sum to 191 min for tier 0, 516 min for tier 1 and 268 min for tier 2, so tiers 0 and 1 carry about 73 percent.
The rubric's deferred-research rule wants that answer recorded with its evidence.

### [LOW] Named-import changes can change the binary's init set

**Location:** `_lym/discussion/decision-record.md` D5, fourth bullet.

**Issue:** Removing a function's last use of a package forces its named import out, and the package's `init` can leave the binary with it.
A test of an importing package that relied on that registration, without covering the changed function, is not selected.
Adding a named import adds an `init` the same way.

### [LOW] Same-named init functions have no defined pairing

**Location:** `_lym/discussion/decision-record.md` D4 and D5, fifth bullet.

**Issue:** `funcKeyName` gives every `init` the key name `init` (`internal/testcover/blockkey.go:185-190`), so "present in both whose body text differs" cannot pair them when their count or order differs.
D5's "adds an init function the base lacks" is likewise undefined when the base already has one.

### [LOW] The D3 criterion contradicts D6's over-budget whole run

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria, first D3 bullet.

**Issue:** The criterion allows nothing outside the union but the same-name overlap.
D6 runs the selected packages whole when the pattern exceeds the command budget.

### [LOW] D7 has no label for two whole-run causes

**Location:** `_lym/discussion/decision-record.md` D7.

**Issue:** A package run whole because its tests use the `lym` binary (D3), and a selected package run whole past the budget (D6), fit none of the four labels the D7 criterion requires.

### [LOW] The D5 new-file criterion is ambiguous

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria, first D5 bullet.

**Issue:** "A new file in a covered package included" fits D5's file-set check only for a file added between the map build and the comparison point.
Read as a file the diff adds, it contradicts D4 and D5, under which a new file of plain functions is no miss.

### [LOW] The PackageBlocks hint points at the wrong data

**Location:** `_lym/discussion/decision-record.md` Notes for the plan writer, last bullet.

**Issue:** `testcover.PackageBlocks` filters `[]ProfileBlock` by directory (`internal/testcover/profile.go:78-87`).
The kept set D2 changes is built from the map's `[]BlockKey` (`internal/gatecli/redundancy.go:592-593`), so the hint sends the plan writer to the wrong data.

### [NIT] The test-speed constraint restates PATTERN text

**Location:** `_lym/discussion/decision-record.md` Constraints, last bullet.

**Issue:** The bullet restates the two PATTERN entries' general rules without a task-specific requirement.
Every run already gets those rules.

## Rejected

- Walkthrough and existing-paths on D6's `recordPass`: green entries take `run.args` per tier run, not per invocation. That is plan plumbing for the end state D6 already names, and no defect in the record.
- Bound-breaker on D6's command budget: `greenCommandBudget` bounds a carried remainder's targets, not a `-run` pattern. Reusing it is a choice the Notes point to, and the size suits a pattern. Not a defect.

Focus checks with no finding: D4's naming premise holds for methods, generic receivers and closures. Package-level declarations, signatures and new methods widen through D5. Every id from D1 to D8 appears as a whole word in Acceptance criteria.
