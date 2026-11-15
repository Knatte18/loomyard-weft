---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: cmd/lym/testonlycode_linux_test.go:47
    summary: The allowlist admits battenrecipe.RecipeEngines and fabricvocab.FailsBareCheck, though neither reads its package's own state nor writes a record, so PATTERN-test-only-code's exception does not cover them
    fix: >-
      Move both out of production and drop their two entries from testOnlyCodeAllowed in cmd/lym/testonlycode_linux_test.go.
      (1) battenrecipe.RecipeEngines: add to internal/testkit/shedcheckkit a function RecipeEngines(source []byte) []string
      that parses source with shedbuild.Parse, panics naming shedcheckkit on a parse error, and returns the rows' distinct engine names sorted
      (the body now in internal/battenrecipe/names.go).
      Delete RecipeEngines from internal/battenrecipe/names.go and drop it from that file's header comment.
      In internal/shedrecipe/coverage_guard_test.go call shedcheckkit.RecipeEngines(recipes.BattenRecipe) in place of battenrecipe.RecipeEngines(),
      and in internal/battenrecipe/recipe_test.go point TestRecipeEngines_ReportsEveryEngineNameSortedOnce and the file header at
      shedcheckkit.RecipeEngines(recipes.BattenRecipe), keeping its assertion against battenRowEngines.
      (2) fabricvocab.FailsBareCheck: delete it from internal/fabricvocab/fabricvocab.go and add it as the unexported failsBareCheck
      with the same body, reading fabricvocab.ConfigsyncOwnerDir, to internal/lymcwd/enforcement_test.go (package lymcwd),
      and change that file's three calls to failsBareCheck.
      The matching summary.md edit is F2's, which the parent makes.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:56
    summary: The change description still describes the round-1 tree, naming an allowlist, a kept helper and follow-up work the diff no longer bears out
    fix: >-
      Leave the code as it is and defer this finding to the run's parent, which owns _lym/landing/summary.md and lies outside the fixer's write surface.
      The parent edits _lym/landing/summary.md:
      under Setup helpers, drop PushAnchored from the fabricengine bullet;
      in the allowlist bullet list, remove the line naming shedcheck.Check with its helpers and shedbuild.Check,
      and add hibernateengine.TerminalPhases;
      delete the sentence saying moving the pure ones, such as the recipe checkers, into a kit is possible follow-up work;
      add a bullet under What changed saying internal/shedcheck moved to internal/testkit/shedcheckkit and shedbuild.Check was deleted,
      and that hubforge.NewRecordsStopError builds the weft stop error for tests outside fabricengine;
      write loomshed's archivedPlanWebsterDirs in lower case, as planwrite_test.go now declares it;
      and once F1 lands, take battenrecipe.RecipeEngines and fabricvocab.FailsBareCheck out of the allowlist list.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/doc.go:396
    summary: The package doc presents SnapshotWarpSHAForTest, a function of export_test.go, as the package's snapshot reader with consumers, though production has no snapshot read path
    fix: >-
      In internal/fabricengine/doc.go, open the snapshot read-side paragraph (line 396) by stating that production code reads no snapshot;
      the Snapshot trailer's one reader is SnapshotWarpSHAForTest in export_test.go, which tests use to check the write path
      over scanWarpSHATrailers.
      Delete the sentence explaining why no snapshot index exists (a snapshot read is rare, a staleness check at a phase boundary)
      and the per-branch paragraph's consumer-regenerates clause (lines 422-428), which describe reads that no production caller makes;
      at line 490 name RebuildIndex alone as the cache-free reader that scans the whole history.
---

### [BLOCKING] Two allowlist entries fall outside the pattern's exception

**Location:** `cmd/lym/testonlycode_linux_test.go:47` (entries for `internal/battenrecipe.RecipeEngines` and `internal/fabricvocab.FailsBareCheck`)

**Issue:** `PATTERN-test-only-code`, as the decision record's parent entry widens it, lets a function only tests call stay in its production file only when no `_test.go` file can serve its tests and no kit can hold it, because it reads its package's own state or writes a record its package alone writes.
I checked every allowlisted function's body.
The others read unexported state or code of their own package (`shedrecipe.registry`, `fabricvocab.sideTokens` and `owners`, `hibernateengine.terminalPhases`, `shortcutcli.shortcutFile`, `loomrecipe.embeddedRecipeSource`, `websterengine`'s unexported names and `fingerprint`), or write a record their package alone writes.
Two do neither:

- `battenrecipe.RecipeEngines` (`internal/battenrecipe/names.go:73`) reads only `recipes.BattenRecipe` and `shedbuild.Parse`, both exported by other packages.
  Its callers are `internal/shedrecipe/coverage_guard_test.go` and battenrecipe's own `TestRecipeEngines_ReportsEveryEngineNameSortedOnce`.
  A kit can hold it: `shedbuild` imports no `*cli` package, `os/exec` or `gitkit`.
  Round 1 already moved the pure recipe checker into `internal/testkit/shedcheckkit` on this ground.
- `fabricvocab.FailsBareCheck` (`internal/fabricvocab/fabricvocab.go:114`) compares its argument with the exported constant `ConfigsyncOwnerDir`.
  An exported constant is readable from any package, so it is not state that bars a move.
  Its only caller is `internal/lymcwd/enforcement_test.go`, a `package lymcwd` test file that can hold it.

Each allowlist entry admitted outside the exception violates the declared rule and is BLOCKING under the constraints section.

### [MEDIUM] The change description describes the round-1 tree

**Location:** `_lym/landing/summary.md:56`, `:72`, `:79`, `:48`

**Issue:** The description makes claims the diff no longer bears out.
- Its Setup helpers list names `fabricengine`'s `PushAnchored`, which commit `61ae9bd1` dropped from `export_test.go`.
  `PushAnchored(` matches nothing in the tree.
- Its allowlist list names `shedcheck.Check` with its helpers and `shedbuild.Check`.
  `internal/shedcheck` is now `internal/testkit/shedcheckkit`, and `shedbuild.Check` is deleted.
- The allowlist list omits `hibernateengine.TerminalPhases`, which `testOnlyCodeAllowed` carries.
- It calls moving the recipe checkers into a kit follow-up work, though `4662725c` made that move.
- It never mentions the new kit or `internal/hubforge/recordsstop.go`'s `NewRecordsStopError`.
- It names `loomshed.ArchivedPlanWebsterDirs` as kept in a `_test.go` file, but the test file declares the unexported `archivedPlanWebsterDirs`.

The round-1 fixer deferred the `PushAnchored` and `shedcheck` edits to the parent because its instructions bar `_lym`, so the same route holds here.

### [LOW] doc.go presents a test-file function as the package's snapshot reader

**Location:** `internal/fabricengine/doc.go:396`

**Issue:** The diff moved `snapshotWarpSHA` out of production into `export_test.go` as `SnapshotWarpSHAForTest`, and the doc now names that function.
The surrounding text still describes it as a live read path with consumers: "a snapshot read is rare, a staleness check at a phase boundary" (line 400), "the consumer regenerates from scratch" (line 424), and "the cache-free readers (`SnapshotWarpSHAForTest`, `RebuildIndex`)" (line 490).
No production code reads a snapshot tag any more.
A reader of the package doc would look for a snapshot reader that only the test build has.
`PATTERN-code-conventions` has a package's `doc.go` document that package, and here it documents a read API the package does not ship.
The round-1 fixer reported that round's F6 fixed with "the consumer sentences no longer describe a production reader", but the three sentences quoted above still do.

## Focus verdicts

- **Allowlist (directive 1).** `internal/shedcheck` is gone: it moved whole into `internal/testkit/shedcheckkit`, and `shedbuild.Check` is deleted.
  Of the remaining entries, F1 names the two that fail the exception; the rest stay under it.
- **Board-worktree refusal (directive 2).** `internal/fabricengine/replay.go:755` builds its request with `ownedBoardWorktree()`.
  `TestReplayRequest_RefusesABoardWorktreeOfNoHub` (`destroy_landedwork_integration_test.go:36`) runs `checkPathRequest(replayRequest(foreign, ownedBoardWorktree(), nil))` on a board-named worktree no hub registers and asserts a `CheckOwnership` refusal.
  The live behaviour is covered; not a defect.
- **`export_test.go` (directive 3).** `PushAnchored` is gone, and both push test files now drive the live `PushPairAnchored`.
  `CommitWeftForTest` reproduces the lock around the live `commitWeftLocked`, and `CodeLymLink` and `WireJunctions` are setup helpers.
  `SnapshotWarpSHAForTest` is a thin reader over the live `scanWarpSHATrailers`.
  Its assertions in `snapshot_integration_test.go:135` and `commit_integration_test.go:263` pin that scan's topological order and `Commit`'s trailer write, so it serves as a reference, not a copy tested as its own subject.
  Not a defect; F3 covers only the package doc's description of it.
- **Stale references (directive 4).** I collected every function the diff deletes from a production file and searched specs, docs, stencils, YAML and Go comments for each name.
  The named examples are fixed: `contracts/specs/shed-recipe-spec.md`, `internal/fabricengine/doc.go`, `internal/discussionparser/append.go`, `docs/overview.md` and `docs/benchmarks/board-performance.md` no longer name a deleted symbol.
  The remaining hits are under `docs/benchmarks/` measurement reports and `docs/research/` records, which record past runs and are exempt.
  Not a defect.
