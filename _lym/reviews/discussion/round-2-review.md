---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 (A build whose listing of the linked packages fails records its failure with no check)
    summary: D6 exempts a failed linked-package listing from both checks, so a dirty edit or a moved HEAD writes a failed-build record onto the map under the start commit, against D6's own rule
    fix: In D6, replace the sentence "A build whose listing of the linked packages fails records its failure with no check, since no linked set is known." with "A build whose listing of the linked packages fails is checked before its failure is written for HEAD and for the files the package itself owns, since no linked set is known; a failed check leaves the map unwritten and lists the package as unmeasured, as below." Add an acceptance criterion with the D6 prefix reading "a package whose uncommitted edit makes the listing of its linked packages fail gets no map written, is listed as unmeasured naming the file, and the call exits non-zero."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 Bound, D3 and D7
    summary: The record never says which time bounds a map run's coverage builds and test processes run under, though the verb reads redundancy_build_timeout_min and redundancy_test_timeout_sec and every tier carries its own timeout_min and test_timeout_min
    fix: In D3, after the sentence ending "under its tier's env (LYM_TEST_TIER included).", add "Each coverage build is bounded by `redundancy_build_timeout_min` and each test process by `redundancy_test_timeout_sec`, as the verb's are today; a build past its bound is a failed build and a test past its bound a timed-out test." In D1's Bound, extend the list of tier settings map mode skips to name the tiers' `timeout_min` and `test_timeout_min`, so it reads "map mode skips the tiers' commands, their `filter`s, their `memory_mb` scopes, their `timeout_min` and `test_timeout_min`, and the shared build step".
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 (replaced whole) and D8 Rationale (never a mix) against D7 (keeps its old entries)
    summary: D2 and D8 say a map is always replaced whole from one measurement, but D7 keeps a failed build's old entries, so that write merges with the map it reads and can mix two runs
    fix: In D2, change "each package in the call is measured whole and its map is replaced whole." to "each package in the call is measured whole and its map is replaced whole, except that a failed build keeps the old entries per D7." In D8's Rationale, change "The last-write-wins race leaves a whole map from one measurement, never a mix, since each write replaces the map at once." to "The last-write-wins race leaves a whole map from one measurement, since each write replaces the map at once; a failed build's write keeps the entries it read, which may come from the other run."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (so the tier's cap bounds the run's test processes together)
    summary: The bound that the tier's memory_mb caps the test processes together fails on a host whose memory cap does not cover a process tree, where one test with children can pass memory_mb
    fix: In D4, change "so the tier's cap bounds the run's test processes together." to "so the tier's cap bounds the run's test processes together on a host whose memory cap covers a process tree; on one where it does not, one test runs at a time and each process is capped alone, as the tier's own `memory_mb` is there."
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 Bound
    summary: D6's checks read tracked files and git status only, so a gitignored Go file in a package directory is compiled into the coverage binary and enters the map under HEAD's hashes unseen, and the Bound does not say so
    fix: Append to D6's Bound the sentence "A gitignored file in a package directory is not seen by either check, so a gitignored Go file the build compiles enters the map under HEAD's hashes."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, the D7 criterion on green skips
    summary: The D7 criterion that a later gate call runs its tiers rather than skipping them as green fails on a tree that already held a green record before the map run, which the gate carry lawfully skips
    fix: Change the D7 criterion "a gate call after it over the same tree runs its tiers rather than skipping them as green" to "a gate call after it over the same tree, with no green record from before the map run, runs its tiers rather than skipping them as green".
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 (the larger of tier 0's and tier 1's slot asks) and the PATTERN-gate-slot constraint
    summary: The larger of the two slot asks does not say that slots all ranks above any count, and the config stores all as 0, so a numeric max picks the wrong value
    fix: In D7's first sentence and in the PATTERN-gate-slot constraint bullet, change "the larger of tier 0's and tier 1's slot asks" to "the larger of tier 0's and tier 1's slot asks, where a tier asking for every slot ranks above any count".
---

### [MEDIUM] A failed listing is written with no check

**Location:** `_lym/discussion/decision-record.md` D6, the sentence on a failed listing of the linked packages.
**Issue:** D6 rules that a map is written only when its files are clean and HEAD is still, then exempts a build whose listing fails from both checks.
D7 writes that failure onto the map: `writeFailed` (`internal/gatecli/testmappackage.go:857-873`) stamps it with the call's HEAD, and replaces a map with no test entries by one recording only the build failure.
Case: an uncommitted edit in package P imports a package that does not exist yet, so `go list` fails;
P's map now records a failed build at a HEAD that builds clean, and `lym gate redundant P` is refused for a map "recording only a failed build" (D9).
The same write goes through after HEAD has moved.
The stated reason, no linked set is known, does not cover the HEAD check or the check of the package's own files, which need no linked set.
The walkthrough and bound-breaker lenses found the same defect independently.
Round 1's fixer added the sentence for a self-check finding that the failed listing never reached a check;
the fix closed that gap by dropping the checks instead of running the ones still possible.

### [MEDIUM] The time bounds of a map run are unstated

**Location:** `_lym/discussion/decision-record.md` D1's Bound, D3 and D7.
**Issue:** Today the verb bounds each coverage build by `redundancy_build_timeout_min` and each test by `redundancy_test_timeout_sec` (`internal/gatecli/testmappackage.go:647`, `:675`; `internal/gateslot/doc.go:177-182`).
Under `lym gate test`, each tier carries `timeout_min` and `test_timeout_min` (`internal/gateslot/tiers.go:34-36`), and the hub's tier 1 sets `timeout_min: 30`.
D1's Bound lists the tier settings map mode skips, but it names neither timeout, and D3's timed-out outcome and D7's failed build both rest on a bound the record never names.
A plan writer moving the loop under the gate could apply a tier's `timeout_min` to a whole `--all` run.
Plan-Write never reads the support log, so the choice has to sit in the record.

### [LOW] D2 and D8 say "whole" while D7 keeps old entries

**Location:** D2's first sentence and D8's Rationale, against D7's failed-build rule.
**Issue:** D7 keeps a failed build's old entries, so its write merges with the map it reads.
D2 says every map is replaced whole, and D8's Rationale says a race never leaves a mix.
Two runs over one package from one worktree, one of which fails its build, can leave the other run's entries beside the first run's failure.
The design choice in D7 stands; D2 and D8 have to name it as their exception.

### [LOW] The tier memory bound holds only where the cap covers a tree

**Location:** D4, "so the tier's cap bounds the run's test processes together".
**Issue:** Where `proc.MemoryCapCoversTree()` is false, `clampParallelToMemory` returns 1 (`internal/gatecli/redundantmutants.go:541-543`) and the per-test cap limits each process alone.
One test that starts children can then hold more than the tier's `memory_mb`.
The tier's own `memory_mb` is equally weak on such a host (`internal/gateslot/template.yaml:15`), so the design is sound, but the bound is stated without its condition.

### [LOW] Gitignored files escape D6's checks

**Location:** D6's Bound.
**Issue:** The package tree is hashed from `CodeTreeAt(rev)` and the dirty check reads `fabric.Status()` (`internal/gatecli/testmaptree.go:28`, `:80`), and neither sees a gitignored file.
A gitignored Go file in a package directory is compiled into the coverage binary, and its coverage enters the map under HEAD's hashes.
The verb behaves the same today; the Bound names the reverted-edit gap but not this one.

### [LOW] The D7 green-skip criterion ignores an earlier green record

**Location:** Acceptance criteria, the first D7 criterion.
**Issue:** The gate carry lawfully skips a tier on a green record for the same tree (`pattern/PATTERN-gate-slot.md`, Gate carry).
A green record from before the map run makes the criterion fail although D7 holds, so the criterion has to state that no such record exists.

### [NIT] "The larger slot ask" leaves `all` unranked

**Location:** D7's first sentence and the `PATTERN-gate-slot` constraint bullet.
**Issue:** `Tier.Slots` stores `slots: all` as 0 (`internal/gateslot/tiers.go:42-43`), so a numeric larger-of picks 1 over `all`.
D7's Bound shows the intended meaning; the rule does not state it.

## Focus verdicts

- D4 with a tier `memory_mb` below `redundancy_test_memory_mb`: the record refuses such a hub in both D1 and D4, naming the tier, both values and `lym config gate`, with a matching criterion, so the quotient is never 0.
  With the refusal in force, the bound holds wherever the cap covers a process tree; F4 records the one host where it does not.
- D3 and the prebuilt `lym`: no test a map run executes reaches a `lym` build.
  `lymbin.prebuilt` is the one reader of `LYM_PREBUILT_LYM` (`internal/testkit/lymbin/lymbin.go:49-59`), `spawnRef` marks every `lymbin` reference (`internal/testcover/spawnscan.go:355`), tier 0 files get an empty verdict but cannot import `lymbin`, and the loop skips `OutOfProcess` and `Unresolved` tests (`internal/gatecli/testmappackage.go:139`).
  Every D3 criterion describes a test that can run: a plain tier 0 or tier 1 test side by side, a spawn-marked test left unrun, and a skipped test.
  Not a defect.
- A package with tier 0 and tier 1 tests: D4 runs tier 0's tests first and then tier 1's, each up to its own `parallel` and its own `memory_mb` quotient, so one tier's count runs at once; the coverage builds take tier 0's `parallel` as `-p` and run outside both caps, as D4's Bound says.
  D1's refusal list covers every `lym gate test` flag but the scope flags, `--main-packages`, `--wait` and the already-refused `--tags`.
  D6 and D7 agree on which maps are written except for the failed-listing write (F1); D7's slot rule leaves `all` unranked (F7).

## Rejected

- The interactions lens's D1/D7 wait-cap finding: `chooseCap` (`internal/gatecli/detachfront.go:466-479`) gives the short cap to every call without a gate key, and its Bound says a mis-pick costs cache money only, never a verdict; a map call waits as any target call does, so the record needs no Decision on it.
