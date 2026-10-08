---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-gitrepo-reset-keep.md
    summary: No card raises the per-package test budget in cmd/lyx/testdata/test-budget.yaml although cards 3, 5, 8, 10, 14, 18 and 20 add top-level tests to packages already at their budget, so TestTestBudget_PackagesStayWithinBudget fails
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-gitrepo-reset-keep.md
    summary: Card 10 adds Repo.ResetKeep, which calls runChecked, but never edits gitrepoPinnedRunBoundMethods in cmd/lyx/gitrepoboundary_test.go, so TestGitrepoBoundary_PinnedRunCallSites fails on the set drift
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/20-plan-write-archives-webster.md
    summary: Card 20 makes planWriteEntry require Env.Rework.ArchiveWebster, but the shared newTestEnv fixture and the PlanWrite row of conformance_test.go leave Rework zero and no card lists them
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-reset-start-archives.md
    summary: The reset's pending-findings guard inherits headBeforeEveryStart, which card 14 rewords to name reset --to start again, so a reset over disjoint start histories with pending findings refuses toward itself
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-reset-start-archives.md
    summary: Card 12 bundles three independently reviewable units, the factored archive and its guard, the archive-only PlanReset path, and the standalone keep-reset in webstercli, and adds exported symbols reused across packages without their own card
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/14-retire-fresh-after-reset.md
    summary: Changed-text sites in internal/webstercli are in no card, the run command long help still says the mismatch refusal names run --fresh, and the reset long help keeps its standalone refuses sentence and its changes no other webster state sentence after card 12
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/12-reset-start-archives.md
    summary: archive_test.go re-pins the record rename and the prompt clearing that TestArchiveRunFile, TestArchiveReportsDir and TestRun_EntryHousekeeping already cover, against PATTERN-test-economy
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/plan/12-reset-start-archives.md
    summary: Card 12 lists the worktree's DirtyPaths for the uncommitted envelope key but names no seam webstercli can call, since Geometry.git is unexported and Git.DirtyPaths is reachable only through it
---

### [BLOCKING] Test budget rows are not raised by any card

**Location:** `_lyx/plan/10-gitrepo-reset-keep.md` (also cards 3, 5, 8, 14, 18, 20)
**Issue:** `cmd/lyx/testbudget_test.go` pins, per package, the number of top-level `func Test…` functions under every build tag, with the row in `cmd/lyx/testdata/test-budget.yaml`.
Its header says a raise is a one-line edit in the same commit as the tests that need it.
I counted each package against its row, and every package below sits exactly at its budget today: `internal/gitrepo` 35/35, `internal/batcher` 8/8, `internal/loomshed` 68/68, `internal/websterengine` 184/184, `internal/planparser` 81/81, `internal/loomcli` 178/178, `cmd/lyx` 68/68.
The plan adds a new top-level test to each:
- card 10, `TestResetKeep` in `internal/gitrepo/reset_test.go`;
- card 3, "one table test over documents" in `internal/batcher/config_test.go`;
- card 5, "a table test" in `internal/loomshed/loompreflight_test.go`;
- card 8, the new `internal/websterengine/merriambase_test.go`;
- card 14, the new `cmd/lyx/websterretiredsteps_test.go` plus its self-test;
- card 18, "one table test" in `internal/planparser/parse_test.go`;
- card 20, `TestPlanCommitPathspec` in `internal/loomcli/wiring_commitstatus_test.go`.
No card lists `cmd/lyx/testdata/test-budget.yaml`, so each of those cards leaves `go test ./...` failing in `cmd/lyx`, which the plan's own `## verify:` runs.
The gap recurs across seven cards, so it is one finding about the method (the writer does not account for the budget file), not seven.
**Fix:** Add `cmd/lyx/testdata/test-budget.yaml` to the Edit targets of every card that adds a top-level test function, raising that package's row by the number added.
Where the behavior fits an existing test as a row (card 3, card 5, card 18), fold it in instead and leave the row alone.

### [BLOCKING] Card 10's ResetKeep drifts the pinned gitrepo run-bound method set

**Location:** `_lyx/plan/10-gitrepo-reset-keep.md`
**Issue:** `cmd/lyx/gitrepoboundary_test.go` (`PATTERN-gitrepo-client-boundary`, a `(test)` entry) compares the set of `*Repo` methods whose body calls `r.run(` or `r.runChecked(` with the literal `gitrepoPinnedRunBoundMethods` list, and fails on any difference in either direction.
The list holds `ResetHard` and no `ResetKeep`.
Card 10 specifies `ResetKeep` as running `git reset --keep <sha>` through `runChecked`, so the computed set gains a member the list lacks and `TestGitrepoBoundary_PinnedRunCallSites` fails.
Card 10 targets only `internal/gitrepo/reset.go`, `reset_test.go` and `doc.go`.
**Fix:** Add `cmd/lyx/gitrepoboundary_test.go` to card 10's Edit targets and say its `gitrepoPinnedRunBoundMethods` list gains `ResetKeep`.
`cmd/lyx/checkedcall_test.go` needs no edit, since `runChecked` is not a raw site.

### [MEDIUM] PlanWrite's new required seam breaks the shared test fixture

**Location:** `_lyx/plan/20-plan-write-archives-webster.md`
**Issue:** Card 20 makes `planWriteEntry` require `env.Rework.ArchiveWebster` through `requireSeam`.
`internal/shedrecipe/fixture_test.go`'s `newTestEnv` leaves `Rework` zero (only `reworkTestEnv` fills it for `PRRework`), and `conformance_test.go` builds its `PlanWrite` row from `newTestEnv`, as do the happy-path and gate tests of `entries_planwrite_test.go` and `entries_gate_test.go`.
The card lists `entries_planwrite_test.go` but neither the fixture nor `conformance_test.go`, so those rows fail construction.
Card 5 enumerates the same kind of call-site fallout for its constructor parameter; card 20 does not.
**Fix:** Add `internal/shedrecipe/fixture_test.go` (fill `Rework.ArchiveWebster` in `newTestEnv`) to card 20's Edit targets, and name the `conformance_test.go` and `entries_gate_test.go` rows it keeps green.

### [MEDIUM] The reset's own guard can refuse toward the reset

**Location:** `_lyx/plan/12-reset-start-archives.md` (with `_lyx/plan/14-retire-fresh-after-reset.md`)
**Issue:** Card 12 runs `freshPendingDrop` for the reset and lists a caller-specific step for four conditions: contract file, differing plan path, differing suspect path, HEAD not at the start.
`freshPendingDrop` has a fifth refusal, `headBeforeEveryStart`, for recorded starts that share no single oldest commit while HEAD is not an ancestor of every one of them.
Card 12 does not give that refusal a reset-caller step, and card 14 rewords it unconditionally to `resetToStartSteps("lyx webster run")`, i.e. `1) lyx webster reset --to start; 2) lyx webster run`.
When the recorded starts have no common ancestor (an archive-only cause in card 12's closed set) and findings are pending, `reset --to start` cannot move and refuses with a way forward that is the same verb.
The `fresh: recorded starts diverge` row card 12 rewrites ("or archiving without moving when none exists") then promises an archive the guard denies.
`PATTERN-refusal-way-forward` requires a way forward that makes progress; a loop back to the refused verb does not.
The case is rare (disjoint histories), but this task exists to remove such dead ends.
**Fix:** In card 12, give `headBeforeEveryStart` the caller's re-run step as it gives the other four, naming `lyx webster accept-audit` (or `accept-audit --batch NN`) for the reset caller, and have card 14 reword only the `run --fresh` caller.
Reuse `restoreStep` for the differing-suspect-path step instead of the literal `git checkout <start> -- <path>`, which cannot restore a path the start does not hold.

### [MEDIUM] Card 12 is three units and hides reused exported symbols

**Location:** `_lyx/plan/12-reset-start-archives.md`
**Issue:** The rubric asks for one card per independently reviewable unit and a card of its own for a symbol reused across packages.
Card 12 carries: (a) the factored in-place archive moved out of `Run`'s fresh branch, with its `freshPendingDrop` guard and caller-specific steps; (b) the archive-only `PlanReset` path with its closed cause set; (c) the standalone keep-reset in `webstercli` with `KeepResetRefusal` and the `uncommitted` envelope key.
(a) is a refactor of `Run` with a second caller, (b) is `PlanReset` behavior, (c) is a CLI and `gitrepo` consumer; each can be built, tested and reviewed alone, and (a) and `KeepResetRefusal` are new exported symbols called from another package.
Card 12 is the largest card (about eleven kilobytes, twelve Edit targets, nine refusal-spec edits) and the one the focus questions concentrate on, which is where a bundled card hides defects (F4, F8).
**Fix:** Split into a factored-archive card (with the guard and `Run --fresh`'s unchanged behavior), an archive-only `PlanReset` card, and a standalone-reset card, in that order, with the refusal-spec rows landing in the card that introduces each refusal.

### [LOW] Changed-text sites in webstercli are in no card

**Location:** `_lyx/plan/14-retire-fresh-after-reset.md`
**Issue:** `internal/webstercli/run.go`'s `runCmd` long help says the command refuses on a fingerprint mismatch "with a message naming run --fresh".
After card 14 the mismatch way forward names `reset --to start` then a plain run, and after card 19 an index-only change names rebaseline; `run.go` is in no card's targets, and card 14 lists only the accept-audit, rebaseline and reset long help.
Separately, `resetCmd`'s long help keeps "In standalone mode it refuses and names the git reset --keep command to run" and "changes no other webster state" after card 12 lands, since card 11 rewrites only its target list and card 14 only its post-reset run step.
The `git reset --keep` literal is therefore pinned only by card 14's new test, so card 12 lands help text that contradicts the behavior.
**Fix:** Add `internal/webstercli#websterCLI.runCmd` to card 14's targets with the reworded mismatch sentence, and have card 12 rewrite `resetCmd`'s standalone and "changes no other webster state" sentences in the same card that changes that behavior.

### [LOW] archive_test.go repeats existing archive coverage

**Location:** `_lyx/plan/12-reset-start-archives.md`
**Issue:** Card 12 has `archive_test.go` pin "the record renamed and the prompts cleared".
`TestArchiveRunFile` and `TestArchiveReportsDir` in `archive_test.go` already pin the rename and stamp helpers, and the `--fresh archives the stale state and reports and clears the prompts` case of `TestRun_EntryHousekeeping` (`runlevel_test.go`) pins the rename of `state.json` and the reports dir and the cleared prompts through the code card 12 factors out.
Only the reset-caller way-forward text of the guard refusal is new.
`PATTERN-test-economy` makes a new assertion for covered behavior a defect.
**Fix:** Limit the new `archive_test.go` coverage to the guard refusing with the reset caller's way forward and leaving the record in place, and leave the rename and prompt clearing to the existing tests.

### [LOW] No seam for listing uncommitted paths from webstercli

**Location:** `_lyx/plan/12-reset-start-archives.md`
**Issue:** The `uncommitted` envelope key is "the worktree's `DirtyPaths` after the move", read in `webstercli`.
`Git.DirtyPaths` is reachable only through `Geometry.git()`, which is unexported, and `Geometry.Git` is nil for the real repository.
The card's `Uses` names `WorktreeChangedFiles` and `ChangedFilesSince` for the keep refusal but nothing for this list, which must exclude the run's own state as accept-audit's dirt check does (`runOwnPath`).
**Fix:** Name the reader: either an exported `websterengine` function over `Geometry` that returns the dirt minus the run's own paths, or `WorktreeChangedFiles` filtered the same way, and list it in the card's `Uses`.

## Verdicts on the focus questions

- **Standalone keep-reset route.** Only `internal/websterengine/gitwrap.go` contains `gitrepo.New(` in non-test code, and it is the one allowlisted file in `cmd/lyx/rawgitmutation_test.go`; no other file there does.
  Card 12 places the call in `internal/webstercli`, which that guard does not scan, so the guard stays green and the engine stays free of mutating git.
  The route is acceptable under D4 and the Constraints; the failure is the pinned method list in `gitrepoboundary_test.go` (F2), which the card does not name.
- **Reset to start and pending state.** Card 12 states the handling for all three paths: the factored archive runs `freshPendingDrop`'s checks against the tree as it stands (after the move for a moving reset), archives only when they pass, carries the drop warnings in a `warnings` key, and a refusal after a move leaves the move and the saved state standing so a re-run converges.
  The refusal rows agree for the moving pair reset and the standalone reset; the one disagreement is the `headBeforeEveryStart` refusal (F4).
- **Refusal-spec rows.** Every row carrying `git reset --keep` or a reset-then-`--fresh` pair is named by card 11, 12, 13, 14, 16 or 19 with a delete or reword, including `audit: fork-state-write`, `audit: parent-write ...` and `audit: fabric-reference` ("refuses toward the `--fresh` route") in card 14.
  The order of two cards rewriting one row is stated for rows 49, 50, 54, 58, 59, 78, 104 and 112; the `reset: target not an ancestor of HEAD` row (cards 11 and 12) is ordered by card 12 referring to card 11's clause.
  No gap found.
- **Changed-text and changed-surface sites.** `persistPlanFingerprintRebaseline` in `cli.go`, the only copy of `PlanFileHashes` outside the engine, is in card 19, and `restampBaseline` and `Run`'s two new-state branches are covered.
  The webster group help's `Verbs:` line is in card 11 through `Command`.
  The tests asserting the old `--fresh` text are in card 14's file list.
  The gaps are the `run.go` and `resetCmd` help sentences (F6).
- **Plan-Write rotation commit.** `CommitPlan` in `internal/loomcli/wiring.go` stages `planparser.PlanDirRel()` only, so the archive additions under `archive-<stamp>/webster/` are covered but the deletions from webster's tracked directory are not.
  Card 20's `planCommitPathspec` adds `websterengine.DirRel()` when that directory or any archived webster directory holds a file, mirroring `reworkCommitPathspec`, so the rotation's commit stages both.
  Not a defect; the one fallout is the fixture seam (F3).
