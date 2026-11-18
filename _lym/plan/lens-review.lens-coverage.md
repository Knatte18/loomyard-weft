```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the plan: each decision, criterion and Intent branch checked for a card and a test row that would fail if it broke.

**BLOCKING**
1. **Card 4 guard vs. cards 6 and 14 (D1, D3, D9).**
   - Card 4's `TestCoverageMapReaderBoundary` fails on any `testcover.ReadMap` call outside `internal/testcover`, `internal/testreach` and three pinned gatecli files: `redundancy.go`, `redundant.go` and `testmappackage.go`.
   - Card 6 has the reach step "read its worktree anchor map" of each changed package.
   - Card 14 plans over "the fresh map's covered blocks".
   - Card 2's Reader API exposes neither a whole `testcover.Map` nor block coverage (`TestsReaching`, `FuncsReachedBy`, `Maps`, `SpawnMarked`, `StaleLinkers`, `Fresh` only).
   - So the natural implementation of cards 6 and 14 calls `testcover.ReadMap` in a new gatecli file. The card 4 guard then fails under card 6's or card 14's gate.
   - Neither card names how the map is read, adds a pinned file, or extends the Reader.

**MEDIUM**
2. **D3 refresh, card 6 `TestGateTestReachStep`.** No row asserts that the refresh actually happens.
   - The rows only pin the non-refresh branches: matching map, dirty own or linked package, past the budget, prime.
   - No row has a package with a stale or absent anchor map refreshed and its anchor map written.
   - The no-map path is untested too: there the linked packages come from `testcover.Toolchain.LinkedPackages`.
3. **D3, card 6, no parent branch.** The Intent adds a "worktree with no parent branch" judged-nothing note. The rows cover red tier, non-Go scoper and outside a hub only.
4. **D3 criterion, "With `darn.gate_fixer` set, a reach turn's result at Darn-Write spawns no cold fixer and reaches the Darn-Write writer's session".**
   - It is tested only in pieces: card 8 `TestWrapRowGate` with a scripted `WriterOnly`, and card 9 `TestWithReachTurn`.
   - Card 9's `TestResolveGateSpec_ValidOneEntryPerVocabularyName` asserts only a non-nil `Rerun`.
   - No row runs the composed path `resolveGateEntry` → `WithReachTurn` → `applyGateFixer`/`WrapRowGate` and checks that no fixer is spawned and the note is returned.
5. **Arbiter ruling (round 2 F1): "A reach note produced by the re-run after a fixer is returned to the writer as the re-run's verdict".** Card 8's Intent states it, but no `TestWrapRowGate` row has the re-run return a `WriterOnly` result.
6. **D4 tier-2 caveat, card 6.** Detection is wired in the step via `testcover.ScanTestFuncs`, but `TestReachReport_NoteLines` uses hand-built reports. No step row has a package holding a tier-2 test, so a broken detection would pass.
7. **D5 criterion, "says no current report … when HEAD's changed functions, their hashes … differ".** Card 6 has one mismatch row: a commit that changes a test file of a named package. No row changes a changed function's text or the set of changed functions.
8. **D7 criterion, "every fixer's report holds the triage lines of its failed tests".**
   - Card 11's ImpactSummary and card 12's Intent argue that the card fixer's report (`detachRun.fixAndRerun`, stderr copied verbatim) carries the lines with no change and no test. Arguing without a test is a finding under this lens.
   - Separately, `execution.printReport`'s wiring has no row: the lookup gated on in-hub, `go` scoper and a failed test, then filling `runFailure.triage`. Only `triageLookup.lines` and `renderFailureReport` are tested in isolation.
9. **D7, two no-change-set causes.** The criterion names two: no parent branch, and an unreadable merge base. `TestTriageLookup_Lines` has one "no change set with its reason" row, and the branch that picks the reason in `printReport` is unreached.
10. **D8, "A worktree with no parent branch walks its current branch".** Card 10 carries it; no `TestGateReach` row reaches it.
11. **D8 / card 10, `--test` naming a test no map holds.** `FuncsReachedBy` returns false. The Intent defines no output for this case, and no row reaches it.
12. **D9 and D11, "`lym gate mutate` refreshes the changed packages' worktree maps".** Card 14's rows cover a stale map not judged and a prime row. No row asserts that a stale package is refreshed and then judged ("one D3 refreshed" is a fresh map).
13. **Card 2, two pass-over branches.** "A map holding only a `BuildFailure`, or one naming a block or function index outside its tables, is passed over" is an Intent branch with no `TestReader` row. Nor does any `TestReaderIndex` row reach the "unreadable index" branch.

**LOW**
14. **D9, unjudged on the memory cap.** The criterion says "unjudged on a timeout or memory cap". Card 14 has a timeout row only.
15. **D9, "until one fails".** No row shows that the covering tests after the first kill are not run.
16. **D9, cap spread.** "`mutation_max_mutants` … spread over the changed functions in turn": card 14's only cap row is "a cap of one runs one mutant", which cannot show the round-robin.
17. **Card 3 branches.** HEAD moved, so no index is written; a failed index write is logged at Warn with the exit unchanged. No row reaches either.
18. **Card 9 smaller branches.**
    - `ReachPendingOf` answering false for a judged-nothing report: no row.
    - A listed-file record error returned as a gate error: no row.
    - The count reset after a pass: no row.
19. **D4 note contents.** The note must name the "commit and age of each map". `TestReachReport_NoteLines` asserts the commits only. Likewise D7's line must name the map's commit, and no `TestTriageLookup_Lines` row states that assertion.
20. **D2 exclusions.** Card 1's `TestChangedFuncs` covers a test file. The other exclusions the Intent lists have no row: a path outside the module, under `testdata`, or under a `.`/`_` directory.
21. **Card 12, webster rerun path.** The Intent sets `Triage` "for the first run and the rerun", but `TestVerifyGate_FailedEvaluationWritesReport` covers one run.
22. **`PATTERN-coverage-map-reader` (test).** The guard pins `ReadMap` and `WriteIndex` callers only. The clauses "finding a diff's changed functions only through `internal/testreach`" and "note used only when it matches HEAD" have no guard.

Tool calls: none; my context held everything this lens needed.
