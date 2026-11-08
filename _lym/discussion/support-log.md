# Support log — redundant-test-finder

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick.

**Turn 1 — exploration.**
Three subagents read `cmd/testtiming`, the coverage study and the gate, Code-Review and dobby code.
Findings: the tool instruments the whole module and runs each test alone, serially; `//testtiming:keep` exists; no advisory gate step or findings channel to Code-Review exists; gate keys hold a `lints` list; dobby counts landings for board tidy; the green record is the only per-worktree gate cache.

**Turn 2 — scope (auto-pick).**
Q: build part 1 alone as a first stage, or all four parts?
Recommended and picked: all four. The goal says redundant tests must stop piling up, which an on-demand tool does not do, and the stencil rules out a staged MVP.

**Turn 3 — tool scope (auto-pick).**
Q: drop the module-wide view or keep it as an option?
Picked: derive both from one module-instrumented run and show the module-wide verdict as a column (D2), so no extra flag or second run.

**Turn 4 — block identity (auto-pick).**
Q: how does a map built at an older commit compare with a diff's tree whose lines moved?
Picked: content-keyed blocks (D3). No git needed to compare blocks; edits err towards no finding.

**Turn 5 — map place and rebuild (auto-pick).**
Picked machine-local under the hub's `.lym`, rebuilt by `lym gate testmap` and by the prime parent's dobby (D4 to D6).

**Turn 6 — step placement and review channel (auto-pick).**
Picked a `reports` list on gate keys and a Go-filled marker in the Code-Burler-Review stencil (D7, D8).

**Turn 7 — lens review.**
Six forks returned reports; every lens returned one. Findings, merged across lenses:

- walkthrough 1, existing-paths 1, interactions 3, bound-breaker 6 (report keyed on HEAD goes stale on every normal path): fixed. D8 matches by judged packages' file hashes and changed tests' function hashes; D7 judges current files; `parent-align` added to the shipped keys.
- interactions 1 (a judged test counts as its own cover): fixed in D7 step 5.
- interactions 2 (any worktree could write the shared map): fixed. D5 refuses outside the prime parent's worktree with a way forward.
- interactions 4, existing-paths 3, bound-breaker 1 (tidy landing count is the wrong trigger): fixed. D6 keeps its own prime-HEAD record and starts on a fresh dobby.
- interactions 5 (dirty skip retried only on the next landing): fixed by stating the bound in D6 and dropping the wrong assumption.
- interactions 6, walkthrough 2, bound-breaker 3 (mutual covers both flagged): fixed. D7 step 5 runs the greedy pass over judged tests.
- existing-paths 2, bound-breaker 4 (no `tier1` env or `lym` on `tier0` keys): fixed. Undeclared tiers are not judged; the step and verb build `lym` once when needed; cost restated.
- bound-breaker 5, existing-paths 5 (a hung test stalls the gate): fixed with a per-test timeout and a step budget.
- bound-breaker 7, board-scope LOW (first refresh holds a slot for the whole module): fixed with `--max` and `test_map_batch`; the slot trade stays stated in D5's bound.
- walkthrough 5, bound-breaker 2 (dependency changes never mark a map stale): fixed with `test_map_max_age_days` and the map's age in the report.
- existing-paths 4 (unknown report name ignored): fixed. D7 refuses it with the known names.
- interactions 7 (red gate and carry-over): fixed. The step runs on carried tiers and writes a judged-nothing report on a red tier.
- interactions 8, walkthrough 4 (D1 criterion ill-defined): fixed by rephrasing.
- walkthrough 3 (D3 claims no git, but step 1 reads base text): fixed. D3 claims only that blocks compare without git; D7 step 1 names the gate's diff route.
- bound-breaker 8 (same-named methods): fixed. D3 includes the receiver type.
- bound-breaker 9 (untracked files): fixed. D5 counts untracked Go files.
- bound-breaker 10 (cap order): fixed. File then name order.
- leanness 1 and 2 (assumptions and constraints repeat decisions): fixed by cutting the repeats.
- leanness 3 (Scope and Goal restate decisions): fixed. Scope In points at D1 to D9.
- leanness 4 (Scope Out bullet about board entries): fixed by cutting it.
- leanness 5 (D4 rationale argues a rejected alternative): fixed.
- leanness 6 (Open risks hold a cost and a non-risk): fixed. The cost is in D5's bound; the `fabric-test-cuts` note is cut, since board-scope found no conflict.
- leanness 7 (D7 cost claim understated): fixed in D7's rationale.
- leanness 8 (PATTERN text inverted): fixed with the suggested wording.

## Rejected alternatives

- Part 1 alone, run as a darn docs task with a cleanup darn: leaves new redundant tests unchecked between runs, which the goal rules out.
- Two tool runs, package-scoped and module-wide: doubles the serial cost for what one module-instrumented run already holds.
- Block identity by position with a git diff line map: needs git to compare blocks and errs both ways; content keys need neither.
- Tracked maps: large, change with every landing, churn history.
- Rebuild in Finalize: a run's Finalize would start or wait on a process that outlives it.
- Rebuild in every dobby, or the verb in any worktree: beam and run worktrees would overwrite maps with their branch's tests.
- Triggering the refresh on the board-tidy landing count: tidy clears it, pauses it and can be off.
- Keying the report on HEAD's commit: every run-record commit and fix commit would make it stale.
- Reporting every contained judged test: two tests covering each other would both be flagged.
- A per-module redundancy test inside the suite: minutes per run and a test binary launching test binaries.
- Failing the gate on a contained test: block coverage does not see assertions.
- Taking the weft write lock for a whole refresh: it would hold every landing into the prime for minutes; D5 rechecks each package's file hashes instead.
- A gate reinterpretation appending `redundancy` to tuned keys: config load already fills the template's `reports` into a tuned `gates` entry.

## Review rounds

**Round 1.** Seventeen findings; F1 disputed, the rest fixed in the record:

- F1 (BLOCKING, the step never reaches a hub whose base `gate.yaml` sets the four keys whole): disputed, premise false. `yamlengine.FillMissing` at config load fills a template key a tuned `gates` entry lacks; D7 now states that, and that `reports: []` turns the step off on a key.
- F2: D7 step 1 reads base text through `fabricengine.CodeFileAt` (new); `AnchoredFileAt` reads the weft, and `Fabric.MergeBase` reads the warp. No parent branch judges nothing.
- F3, F4, F5, F6: D5 lists skips apart from capped leftovers, treats an untracked file as a skip, rechecks each package's file hashes after its tests, and gives each test the step's timeout; D6 retriggers on capped leftovers only.
- The fixer's self-check added: a non-`go` scoper judges nothing, an error-bearing report never skips a repeat call, and a dobby stop ends the refresh child.
- F7 to F14, F16: D7's bound, failure rule, repeat skip, cost, keep escape, drop order, changed-files line and cover order restated.
- F10: the refresh child registers under `runproc.HubOwner` as `test-map`.
- F13, F17: D8 never matches a red report and states the map-rebuild bound.
- F15: the in-place landing moved to Constraints; `merge.go` runs `MergeStart` in the parent's warp checkout under the weft write lock.

**Round 2.** Ten findings, all fixed in the record:

- F1 (BLOCKING, a package with nothing to build had no map and looped D6): D5 maps such a package with no tests and counts it against `--max`.
  The self-check added that a failed-build map is retried after a HEAD move, listed at exit, and not judged by D7.
- F2: the prime dobby's stop calls `runproc.EndRole` with `runproc.RoleTestMap`; `EndRole`'s contract spares other roles.
- F3: the report records every step 1 test's function hash, and D8 matches against that set; file hashes stay per judged package.
- F4: tests run in their own process group with a bounded wait; builds run under `redundancy_build_timeout_min`; D5's time bound excludes the slot wait.
- F5: D7 step 1 looks tests up by package and name, so renames take no unchanged test.
- F6 to F10: D3's ordinal bound, D5's non-Go-file bound, D7's false-cover bound and per-package map age, the spawn scan's move to `internal/testcover`, and the unique criterion restated.

**Round 3.** Five findings, all fixed in the record:

- F1: D7 step 4 takes one gate slot as a non-tier holder, its wait unbounded; `redundancy_budget_min` counts builds from the slot, and no build or test starts once it is spent.
- F2: D5 writes an exit record with its process ID and the capped count; D6 reads it after the wait, and a killed refresh or an unmatched record counts as none left.
- F3: the keep scan moves into `internal/testcover` with the spawn scan.
- F4: D4 hashes the Go files the directory tracks at HEAD.
- F5: D5's tracked-file and status reads go through `internal/fabricengine`; Constraints name `PATTERN-fabric-boundary`.

## Question ledger

- Q1 Stage the work or build it whole? — Resolved, auto-pick: whole (Turn 2).
- Q2 Keep the module-wide view? — Resolved, auto-pick: as a column from the same run (Turn 3).
- Q3 How to compare across commits? — Resolved, auto-pick: content-keyed blocks (Turn 4).
- Q4 Where maps live and who rebuilds them? — Resolved, auto-pick: hub `.lym`, `lym gate testmap` in the prime worktree, prime parent's dobby on a prime HEAD change (Turns 5 and 7).
- Q5 Which gate keys run the step, and how Code-Review sees it? — Resolved, auto-pick: `reports` list on `webster`, `parent-align`, `darn-write` and `code-burler-fix`; a stencil marker matched by content (Turns 6 and 7); a hub that tunes those keys gets `reports` through config load's template fill (Round 1).
- Q6 Limits on the step's cost? — Resolved, auto-pick: `redundancy_max_tests` 40, `redundancy_test_timeout_sec` 120, `redundancy_budget_min` 10 (Turn 7), and `redundancy_build_timeout_min` 10 per build, shared with the refresh (Round 2).
- Q7 Limits on the refresh? — Resolved, auto-pick: `test_map_batch` 5 packages per call, `test_map_max_age_days` 7 (Turn 7), and `redundancy_test_timeout_sec` per test (Round 1), and `redundancy_build_timeout_min` per build (Q6, Round 2).
