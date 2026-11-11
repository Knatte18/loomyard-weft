---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D8 (test_map_every migration and its Rationale)
    summary: A hub that set test_map_every to 0 gets upkeep passes after a binary upgrade until a start verb runs the reconcile walk, because the re-exec'd prime dobby loads beam with the retired key dropped and test_map_upkeep defaulting to true
    fix: In D8, add how the off switch holds before the walk has run. Either beam's load reads a stack still holding test_map_every 0 as test_map_upkeep false (beam's Migrate applied in memory before configengine drops the retired key), or the --upkeep verb and the dobby's upkeep step run hubreconcile.Ensure before reading the key. Add a D8 acceptance criterion that a prime dobby re-exec'd onto the new binary over a beam.yaml holding test_map_every 0, with no walk run, starts no pass.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1 (Miss bullet) and D10 (per-round write)
    summary: A whole rebuild after a Miss writes the map at H after each round, and the record never says that write drops the old entries, so stale entries could be carried to H and read as Current by the next plan
    fix: In D1's Miss bullet or D10's per-round write bullet, state that a whole rebuild's first written round replaces every entry of the map, so the map written at H holds only entries measured at H, and the tests not yet measured are gaps. Add to the D1 Miss acceptance criterion that after a whole rebuild's first round the map at HEAD holds no entry measured before HEAD.
  - id: F3
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D8 (second bullet, self-registration) and D11
    summary: D8 moves the RoleTestMap registration into the verb but says nothing of a failed registration, and an unregistered dobby-started pass blocks an anchor move for its whole run, escapes D11's end and lets the dobby start more passes
    fix: In D8's second bullet, add that a --upkeep call whose own runproc registration fails exits before it plans, and that the dobby starts no pass while a pass it started is still its live child. Drop the Bound clause that accepts a pass started beside an unregistered one, or narrow it to a no-package hand call without --upkeep.
  - id: F4
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (second bullet, lock retake after a yield)
    summary: After a yield the pass takes the build lock again with the slot, but the record does not say the retake never blocks, so a pass can hold a slot waiting on a hand call's lock and a gate then waits past the one-poll bound
    fix: In D9's second bullet, state that the retake tries the package's build lock without waiting, and that when the lock is held the pass releases the slot and waits under D9's rule again.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Open risks, first bullet; support-log.md Question ledger Q15
    summary: The Open risks line asking the parent whether testmap-cross-package's run is held is settled by the parent's Added after Discussion entry, which pauses that run until this record is approved and D13's changes reach it
    fix: Drop the first Open risks bullet of decision-record.md. In support-log.md, mark Q15 resolved, citing the Added after Discussion entry of 2026-11-11.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (Current and Affected bullets) against D4 (outcome-entry drop)
    summary: D1 says a Current verdict keeps every entry and Affected keeps every unaffected one, while D4 drops an outcome entry whenever one of the package's own files differs, which can happen under either verdict
    fix: In D1's Current and Affected bullets, except the outcome entries D4 drops, for example by appending to each that an outcome entry D4 drops is dropped and queued.
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (test_map_upkeep key)
    summary: D8's migration writes test_map_upkeep into beam's local override file, but the record does not classify the key, and only a CategoryNone key loads from the local layer
    fix: In D8's sixth bullet, state that test_map_upkeep is classified configengine.CategoryNone, as test_map_every is today.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, first D8 bullet
    summary: The D8 criterion that an exit record naming queued work starts a pass on the next tick has no slot condition, so it contradicts D8's rule and the one-slot criterion on a hub with fewer than two slots
    fix: Qualify the first D8 acceptance criterion with on a hub with two or more slots.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (marker paragraph) and D6
    summary: D1 says which writes set the marker's Commit and BuiltAt, and D6 puts the environment stamp in the same marker, but no decision says a rebuild writes the current stamp, so a stamp miss could recur at every landing
    fix: In D1's marker paragraph, add that every write that moves a map's Commit also sets the marker's environment stamp (D6) to the current one.
---

The record holds up on each question the focus names except D8's off switch, which fails before the walk runs (F1).
None of the findings is BLOCKING.

The four focus directives:

- D8's migration order holds.
  `configsync.ReconcileHubWideAt` runs `Migrate` before `pruneSparse` in its present-file branch.
  `sparseSeed` runs it on a prime-seeded absent file.
  `hubreconcile`'s `reconcileHubWide` calls it before `applyReinterpretations`.
  For the prime's `beam.yaml` and the local override, `configsync.ReconcileAll` goes through `reconcileSparseFile`, which runs `Migrate` before the prune.
  Two gaps remain: the window before any walk (F1) and the new key's category (F7).
- D1's gap queue drains the backlog after an ended pass, a discarded round or a dropped outcome entry without a next Affected verdict.
  Current and Affected both queue declared tests with no entry, and a Miss rebuilds whole.
  An ended pass leaves its trigger in force.
  A dirty discard costs at most one restart, which the next dirty read skips.
  The open parts are what a whole rebuild's per-round write keeps (F2) and the D1/D4 wording (F6).
- On a hub with fewer than two slots, no pass restarts at the same HEAD.
  A yielded pass and a hand call never deadlock, since a pass holds at most one build lock and gives it back on a yield.
  The `--upkeep` flag tells a dobby-started pass from a hand call.
  D8 bounds the flag: it refuses package arguments, refuses to start beside a live pass and changes only registration and the exit record.
  The open parts are a failed self-registration (F3) and a lock retake that may hold a slot (F4).
- D11 holds against the code.
  `dobbyOwner` maps the prime to `runproc.HubOwner`, and `EndDobbies` calls `runproc.EndSparing` over that owner sparing only push and sync, so it ends a registered pass.
  The pass is the dobby's direct child, so sparing its record in `dobbyPassInFlight` takes it out of the anchor wait.
  D10's parallel rule does select tests: PATTERN-test-speed opens every test with `tiergate.Require`, and tests such as `internal/gateslot/pool_test.go`'s follow it with `t.Parallel()`.

### [MEDIUM] D8's off switch fails between a binary upgrade and the reconcile walk

**Location:** `_lym/discussion/decision-record.md` D8, sixth bullet and Rationale.

**Issue:** `Migrate` runs only inside a reconcile.
The binary-change walk runs only from `hubreconcile.Ensure`, whose callers are `lym beam start` (`internal/beamcli/cli.go:506`), adopt's check, clone and battencli's pair wire.
The prime dobby re-execs onto a changed binary between ticks and reconciles nothing (`internal/dobbyengine/doc.go:113`, `internal/dobbyengine/loop.go`).
Its strict `LoadConfig` goes through `configengine.Load`, which drops retired keys silently (`internal/configengine/config.go:274`, `DropRetired`).
`test_map_upkeep` is absent from every layer and fills from the template as true, so the dobby starts passes on a hub that set `test_map_every: 0`.
This lasts until someone runs a start verb.
D8's claim that the migration "keeps a hub that set it off, in any layer that set it" holds only after a walk, and the record neither closes the window nor bounds it.
Three lenses found this independently.

### [MEDIUM] A whole rebuild's per-round write may carry stale entries to H

**Location:** `_lym/discussion/decision-record.md` D1 Miss bullet; D10 third bullet.

**Issue:** D1's Miss leaves the map as it is and queues a whole rebuild, and D10 writes the map after each round of finished runs.
The record never says whether that first write at `H` starts from an empty map or merges into the old one.
A merge would carry entries the Miss declared untrusted to `H`.
D9's re-plan after a yield, or the next pass's plan, would then read the map as current at `H` and keep them.
The selector would skip tests the Miss had marked unknown, and only the landing gate at scope `all` would catch it.

### [LOW] A failed self-registration leaves an unbounded pass outside D11

**Location:** `_lym/discussion/decision-record.md` D8 second bullet and Bound; D11.

**Issue:** The dobby still starts the pass as its child (`internal/beamcli/dobbydeps.go`, `startTestMap`).
D8 moves the `runproc.RoleTestMap` registration into the verb and says nothing of a failed registration; today's precedent leaves an unregistered refresh to run.
`dobbyPassInFlight` (`internal/fabriccli/anchordeps.go`) counts every unspared live child of the dobby, and D11 spares only a registered pass.
`runproc.EndSparing` ends only recorded processes.
An unregistered pass therefore holds an anchor move for its whole run, which D8's lifted batch cap leaves unbounded.
It also stays invisible to `TestMapRunning`, so the dobby starts further passes.
Two lenses found this.
It is LOW because only a failed write under the runproc directory leads to it.

### [LOW] D9's lock retake after a yield may wait while holding a slot

**Location:** `_lym/discussion/decision-record.md` D9 second bullet and Rationale.

**Issue:** "It takes that lock again with the slot" leaves open whether the retake blocks.
Take a three-slot hub where a hand call holds a slot and package P's build lock.
A pass that yielded on P takes a free slot and then waits on P's lock, and the yield check does not run during that wait.
A gate then takes the last free slot, and a second gate waits for the hand call's whole job.
That breaks the Rationale's "at most one poll behind upkeep".
Today's lock take is the non-blocking `TryAcquireWriteLock` (`internal/gatecli/testmap.go`, `lockPackage`), so a non-blocking retake fits the code.

### [LOW] An Open risks line the parent's decision already settles

**Location:** `_lym/discussion/decision-record.md` Open risks, first bullet; `support-log.md` Q15.

**Issue:** The line asks the parent whether `testmap-cross-package`'s run is held and whether D13's changes reach it before its writer writes code.
The `Added after Discussion (parent, 2026-11-11)` entry answers both: the run is paused at Run-Shed until this record is approved and D13's changes for it are carried.
The rubric classes such a line as `consistency`, and its fix drops the line.

### [LOW] D1's keep rule and D4's outcome-entry drop disagree

**Location:** `_lym/discussion/decision-record.md` D1 Current and Affected bullets; D4 second bullet.

**Issue:** D1 keeps every entry under Current and every unaffected entry under Affected.
D4 drops an outcome entry whenever one of the package's own files differs between `Commit` and `H`.
D2 compares own Go files by function, so a change to an own function no test covered gives a Current verdict while D4 drops the outcome entry.
The D4 acceptance criterion follows D4, so D1's text is the outlier.

### [LOW] test_map_upkeep's config category is unstated

**Location:** `_lym/discussion/decision-record.md` D8 sixth bullet.

**Issue:** D8 migrates `test_map_every: 0` into `test_map_upkeep: false` in beam's local override file.
PATTERN-config-layers closes every key with a category other than None to the local layer, and beam's `LoadConfig` is strict.
A plan that classifies the new key otherwise makes every migrated local file refuse to load.
`test_map_every` is `configengine.CategoryNone` today (`internal/dobbyengine/config.go`, `ConfigSpec`).

### [LOW] The first D8 criterion lacks D8's slot condition

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria, first D8 bullet.

**Issue:** The criterion says the dobby starts a pass on its next tick when the exit record names queued work and no landing came since the last pass.
D8 starts no pass on queued work alone while the slot count is below two, and the one-slot criterion further down says the same.
On a one-slot hub the two criteria contradict.

### [LOW] No decision says a rebuild writes the current environment stamp

**Location:** `_lym/discussion/decision-record.md` D1 marker paragraph; D6.

**Issue:** D1 has every `Commit`-moving write set the marker's `Commit` and `BuiltAt`.
D6 puts the environment stamp in the same marker and makes a differing stamp a miss.
No decision says which write sets the stamp to the current one.
If a whole rebuild left the old stamp in place, every later plan would miss again and rebuild the package at each landing.
