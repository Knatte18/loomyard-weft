---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (last bullet) and Constraints, PATTERN-config-layers bullet
    summary: The reconcile walk prunes retired keys from the hub base file before it applies reinterpretations, so D8's reinterpretation never sees test_map_every 0 and a hub that switched maps off gets upkeep switched on.
    fix: In D8's last bullet, replace the configreg.Reinterpretation with the beam module's configreg Module.Migrate, which runs before the retired-key prune in configsync.ReconcileHubWideAt; state that Migrate rewrites test_map_every 0 to test_map_upkeep false and drops any other test_map_every value. Name the evidence in D8 - hubreconcile walk.go runs ReconcileHubWideAt (pruneSparse drops units a retired key covers) before applyReinterpretations. Change the Constraint's last clause to say test_map_every's 0 is carried by D8's migration, and change the D8 acceptance criterion's wording from reinterpretation to migration where it names the mechanism.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (verdict list) with D4, D8 and the D4 criterion on a yield-killed test
    summary: Only an Affected verdict queues a declared test with no entry, so a gap left by an ended pass, a discarded round or a D4 outcome-entry drop is never queued again once the next plan reads Current, and the backlog does not drain on its own.
    fix: In D1, state that under every verdict except Miss the plan queues each declared mapped test of the package that has no entry, spawn-marked tests excepted (D4); a Current verdict whose map has such a test re-stamps the map and queues those tests. In D4's outcome-entry bullet, state that dropping an outcome entry queues that test. Keep the D4 criterion that a yield-killed test is queued again, and add to it that the next pass after the one that ended still queues it.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (last bullet and Bound)
    summary: The hub-walk rewrite reaches only the hub base file, but beam is a per-worktree module whose prime worktree file and local override file can also hold test_map_every 0, which a load then drops as retired with no bound stated.
    fix: In D8, state that the migration also runs on the prime worktree's beam.yaml and on beam's local override file in the reconcile of those layers, rewriting test_map_every 0 to test_map_upkeep false there; or, if the design keeps the rewrite to the hub base file only, add to D8's Bound that a test_map_every 0 set in the prime's worktree or local layer is dropped and upkeep starts, and that the operator sets test_map_upkeep false by hand.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (queued-work bullet) and D9 (fewer-than-two-slots bullet)
    summary: On a hub with fewer than two slots every pass ends with its queue unworked, the exit record names queued work, and D8 starts a new pass on every dobby tick, the restart loop D8's carve-outs exist to prevent.
    fix: In D8's queued-work bullet, add that a pass on a hub with fewer than two slots records no queued work, or that the dobby starts no pass on queued work alone while the hub's slot count is below two; add a D8 criterion that on a one-slot hub a pass with queued work starts no second pass at the same HEAD.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (yield bullet and hand-call bullet)
    summary: A yielded pass that keeps its package's build lock while it waits for two free slots, and a hand call that keeps its slot while it waits for that lock, wait on each other on a two-slot hub until a landing.
    fix: In D9's yield bullet, state that a yielding pass releases the build lock of the package whose job it interrupted and takes it again with the slot; add to D9's Bound that a hand call's wait for a build lock then ends within one pass job. Add a D9 criterion that a hand call naming a package whose job a pass yielded gets the lock without waiting for the pass to retake a slot.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 (second bullet) and the first D10 criterion
    summary: Every Test function opens with tiergate.Require per PATTERN-test-speed, so no test's top-level body opens with t.Parallel() and D10's side-by-side rule selects nothing.
    fix: In D10's second bullet and its criterion, replace "opens with t.Parallel()" with "calls t.Parallel() as a top-level statement of its body before any other statement except tiergate.Require", citing PATTERN-test-speed's rule that every test opens with tiergate.Require.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D11
    summary: D11 puts the end of a pass in the dobby, but the wait lives in lym fabric anchor, whose passesInFlight loop counts the refresh as a live dobby child and whose EndDobbies already ends hub-owner processes once the wait is over.
    fix: Rewrite D11 to state that the anchor verb's in-flight check (fabriccli dobbyPassInFlight) no longer counts a registered runproc.RoleTestMap process as a pass in flight, so the move does not wait for it and the verb's existing end of the prime's hub-owner processes (EndDobbies through runproc.EndSparing) ends it; drop the claim that the dobby ends it.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (exit-record bullet) and D9 (hand-call bullet)
    summary: The record never says how the verb tells a dobby-started pass from a hand call once --max is retired, nor which rules a bare lym gate testmap naming no package follows, and the marker it implies is a new edge with no stated bound.
    fix: In D8, name the marker that makes a call a pass, for example a hidden flag the dobby passes, and state what it lets a hand caller do (start a pass, write the exit record) and what bounds that (the build lock per package, one live pass read through runproc). In D9, state that a call naming no package and lacking the marker runs as a pass but leaves the exit record as it was, or else state the rule it follows instead.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 (non-Go bullet), D3 (last sentence) and the second D2 criterion
    summary: D3 hashes the package's own non-Go files by the linked-package rule, which covers the directory and go:embed targets only, so an edit to a non-embedded file under testdata is no miss, contradicting the D2 criterion that calls it one.
    fix: In D3's last sentence and D2's non-Go bullet, state that a package's own non-Go files include every tracked file under its directory tree down to the next package directory, testdata included; or drop the testdata clause from the D2 criterion and add to D1's Bound that a testdata change carries entries until the audit.
  - id: F10
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks (first bullet) and D13
    summary: Board entry testmap-cross-package shows running at Darn-Write while this task's entry says the build tasks stay held until this record is approved; whether that run is held, and whether D13's changes reach its stored record before its writer writes code, is a parent question.
    fix: Parent - confirm that the testmap-cross-package run is held at Darn-Write, or hold it, until D13's change for that entry is carried onto its stored record; then the writer drops the Open risks line and Q15 in the support log.
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6
    summary: The environment stamp digests the mapped tiers' command and env only, but testmap's test environment also carries the hub env.yaml variables (gateEnv in internal/gatecli/testmap.go), whose change reaches tests with no miss.
    fix: In D6's stamp list, add a digest of the env.yaml variables the call passes its children, and add the env.yaml case to the D6 criterion.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (lock-held skip) and D8 (start rule)
    summary: D1 says the next pass plans a lock-skipped map again, but with no landing D8 starts a next pass only when the exit record names queued work, and the record does not say a lock-skipped map counts as queued.
    fix: In D8's start bullet or D12's field list, state that a map the plan skipped for a held lock counts as queued work in the exit record.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D7 (self-check bullet)
    summary: D7 requires a kept entry's linked functions to appear in H's block table, but D3 stores no block of another package, so the stored table cannot hold them.
    fix: In D7's self-check bullet, state that own-package block keys are checked against the fresh build's block table and linked functions against the fresh build's function set per linked package.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5, D8 (carve-out bullet) and D12 (queued count)
    summary: D8's failed-at-HEAD carve-out does not say it covers D5's failure-only map as well as the new failure field, and D12's queued package count does not say whether it is the plan's count or the work left at exit, which D8's trigger needs.
    fix: In D8's carve-out bullet, state that a recorded failure is either D5's failure field or a failure-only map's BuildFailure with its Commit; in D12, state that the queued package count is the queued work left when the pass ended.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D13 (carrier paragraph)
    summary: D13's carrier sentence covers the three build tasks' stored records and Darn-Write, but two of its five entries, testmap-backlog-drain and redundancy-sweep-existing, are notes with no stored record or Darn-Write writer.
    fix: In D13, add one sentence that the parent carries the change for each of the two notes into the note's body.
---

### [BLOCKING] D8's reinterpretation never sees the retired key

**Location:** `_lym/discussion/decision-record.md` D8 and the PATTERN-config-layers Constraint.

**Issue:** `reconcileHubWide` runs `configsync.ReconcileHubWideAt` (`internal/hubreconcile/walk.go:232`) before `applyReinterpretations` (`walk.go:254`).
For a present hub base file, `ReconcileHubWideAt` calls `pruneSparse` (`internal/configsync/configsync.go:1157`), which removes every unit a retired key covers even without `PruneUnknown` (`configsync.go:719-739`).
The Constraint makes `test_map_every` a retired key, so it is gone before the rewrite reads the file.
`Module.Migrate` runs before the prune (`configsync.go:1140`) and is documented for rewriting retired keys (`internal/configreg/configreg.go:77-78`).
Three lenses found this independently, and I read the walk order myself.

### [BLOCKING] Gaps under a Current verdict are never queued again

**Location:** `_lym/discussion/decision-record.md` D1, with D4, D8 and the D4 criterion.

**Issue:** D1 queues "every declared mapped test with no entry" only under Affected tests.
Suppose a pass drops T at H1 and writes the map at H1, and then a landing ends the pass (D8).
The next plan finds the map Current at H2 and re-stamps it, and T stays unmeasured until something affects a test of that map.
The same happens to a round D10 discards and to an outcome entry D4 drops on an own-file change.
At this hub's landing rate, passes often end early, so these gaps pile up.
This breaks D8's claim that a backlog drains on its own, and it breaks the D4 criterion beyond a single pass.

### [MEDIUM] The off switch in the prime's worktree or local layer is lost

**Location:** D8.

**Issue:** `applyReinterpretations` rewrites only `configengine.ConfigFile(boardDir, module.Name)` (`internal/hubreconcile/reinterpret.go`).
beam is per-worktree (`internal/dobbyengine/config.go` `ConfigSpec`).
The dobby's load also stacks the prime's `_lym/config/beam.yaml` and the local file (`internal/configengine/layers.go:135-146`), and `test_map_every` is `CategoryNone`.
No such file holds the key on this hub today.

### [MEDIUM] One-slot hub restarts passes on every tick

**Location:** D8 and D9.

**Issue:** D9's pass on a hub with fewer than two slots plans and ends with its queue unworked.
D8 starts a pass whenever the exit record names queued work, and its carve-outs leave this case out.
`slots` is a machine key (`internal/gateslot/config.go:148`), so a machine can lower it.

### [MEDIUM] Hand call and yielded pass wait on each other

**Location:** D9.

**Issue:** `holdSlot` keeps the call's slot to the end of the call (`internal/gatecli/testmappackage.go:113-139`).
Take a hand call naming A and P on a `slots: 2` hub, with a pass that yielded mid-job on P.
The hand call builds A, then waits for P's lock while holding its slot.
The pass keeps P's lock, since nothing says it releases it, and waits for two free slots.
Only a landing breaks the wait, and D13 makes this hand call the redundancy sweep's step.

### [MEDIUM] No test opens with `t.Parallel()`

**Location:** D10.

**Issue:** PATTERN-test-speed has every test open with `tiergate.Require(t, N)`.
`internal/gateslot/pool_test.go:40-42` and `internal/testcover/coveragemap_test.go:16-17` show `t.Parallel()` as the second statement.
Read literally, the rule selects no test.

### [MEDIUM] D11 names the wrong actor

**Location:** D11.

**Issue:** The wait is the anchor verb's `passesInFlight` loop (`internal/fabriccli/anchor.go:448-460`).
That loop counts any live dobby child other than push and sync (`internal/fabriccli/anchordeps.go` `dobbyPassInFlight`).
After the wait, the verb's `EndDobbies` (`anchordeps.go:255-268`) already ends the hub-owner processes through `runproc.EndSparing`.
As written, D11 sends the implementer into the dobby, and the move would still wait.

### [MEDIUM] Dobby pass and hand call are not told apart

**Location:** D8 and D9.

**Issue:** Today the dobby passes only `--max` (`internal/beamcli/dobbydeps.go` `startTestMap`), and D8 retires it.
D8's rule that only a dobby-started pass writes the exit record therefore needs a new marker, which the record neither names nor bounds.
A bare `lym gate testmap`, documented in the verb's examples, falls under neither D8's pass nor D9's hand call.

### [MEDIUM] testdata criterion contradicts D3's rule

**Location:** D2, D3 and the D2 criterion.

**Issue:** D3 hashes own non-Go files "by the rule it uses for a linked package's".
The `testmap-cross-package` entry defines that rule as the directory's files plus `go:embed` targets in subdirectories.
A non-embedded file under `testdata` is outside that rule, yet the criterion calls its edit a miss.

### [MEDIUM] A build task runs at Darn-Write before this record is approved

**Location:** Open risks, first bullet, and D13.

**Issue:** `lym board get` shows `testmap-cross-package` as `running · Darn-Write`, while this task's entry says the build tasks stay held until this record is approved.
Its worktree has no commit past main yet.
Only the parent can hold that run and decide when D13's change reaches it.

### [LOW] env.yaml variables are outside the stamp

**Location:** D6.

**Issue:** `prepare` builds the children's environment with `gateEnv(r.request.environ, hubEnv)` (`internal/gatecli/testmap.go:265`).
That environment includes `env.yaml`, which D6 does not digest.

### [LOW] A lock-skipped map has no next pass

**Location:** D1 and D8.

**Issue:** With no landing, D8 starts a pass only on queued work, and nothing says a lock-skipped map is queued work.

### [LOW] D7's self-check names a table D3 does not keep

**Location:** D7.

**Issue:** D3 stores linked coverage as function sets and no block of another package, so "H's block table" cannot hold linked functions.

### [LOW] Failure carve-out and queued count are ambiguous

**Location:** D5, D8 and D12.

**Issue:** D8's carve-out names "a package whose recorded failure names the current HEAD" but not D5's failure-only map.
D12's queued count could mean the plan's count or the work left at exit, and D8's trigger needs the latter.

### [LOW] D13's carrier skips the two notes

**Location:** D13.

**Issue:** The carrier names stored records and Darn-Write.
`testmap-backlog-drain` and `redundancy-sweep-existing` are notes with neither.

## Rejected

- Bound-breaker rated the one-slot restart loop and the hand-call deadlock BLOCKING. I kept both at MEDIUM. Each loses no map soundness, and a landing or a two-slot hub bounds each.
- Walkthrough marked D11's purpose met. The existing-paths lens showed the wait runs in the anchor verb, so I kept F7.
