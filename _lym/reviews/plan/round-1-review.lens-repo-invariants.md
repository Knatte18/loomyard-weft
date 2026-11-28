```yaml
lens: repo-invariants
outcome: reported
```

**Lens: repo-invariants.** I checked each card for test seams, tier tags, doc text it falsifies, and card order.

**Per card, each invariant checked:**
- Card 1:
  - test seam: `Snapshot.Seams` is a map, and `Build` clones it through the card's own `fabrickit#Builder.Build` edit. `TestBuilder` holds only when its `everySetter` (`internal/testkit/fabrickit/builder_test.go:19`) gains the `Seam` setter, which the card's `TestBuilder` row covers. Held.
  - doc: the `fabriccore` and `fabrickit` `doc.go` files are extended. Held.
  - order: no earlier dependency. Held.
- Card 2:
  - tier: one tier0 test. Held.
  - doc: the `shedengine` `doc.go` is extended. Held.
  - order: held.
- Card 3:
  - test seam: the parallel `TestSnapshotReader_ReadsAHub` row only reads seams, and sequential tests run first. Held.
  - doc: the `fabricengine` `doc.go` reader-field list is extended, so held. `TestPersistentPreRunE_InstallsSeams` is edited. Held.
  - order: held.
- Card 4:
  - order: broken in substance. Its run-lock adapter check runs before card 11 moves `tryPrimeLock` onto the seam (finding 3).
  - visibility: `TestInstallRequiredSeams`'s planting helper is not reachable from `fabriccli_test` (finding 4).
- Card 5:
  - doc: the `fabriccore` `doc.go` is extended. Held.
  - deleting `wayforward_test.go` removes only rows the card says the core holds. Held.
  - order: held.
- Card 6:
  - doc and order: held.
- Card 7:
  - gate: broken, the `TestPushClassified_Snapshot` import cycle (finding 1).
  - doc: broken, the class list in `fabricengine/doc.go:651-652` (finding 5).
  - PATTERN-fabric-boundary background clause: edited. Held.
  - `TestBoltReplay_NamesNoBoardFile` readers map: `goneEntryEnded` stays in `pushclassified.go`. Held.
- Card 8:
  - doc and order: held.
- Card 9:
  - gate: broken, the `TestReconcileOrigin_Snapshot` import cycle (finding 1).
  - `remoteRefsLockFetchAllowlist` reason: edited. Held.
  - `EntryFactsForTest` and `ClassifyEntryForTest` go with their last user (`originreconcile_integration_test.go:691-713`), which the card deletes. Held.
- Card 10:
  - doc and order: held. Card 9's pre-lock refusal runs before `recordedRuns`.
- Card 11:
  - doc: broken, the `uncontainedWriteAllowlist` reason text (finding 6).
  - readers map (`addLocalCreatedStamps`): held.
- Card 12:
  - gate: broken in the moved tests' compile (finding 2).
  - tier: the moved tests keep tier1. Held.

**Findings**

1. **BLOCKING · design** — `_lym/plan/07-push-side-core.md` (`TestPushClassified_Snapshot`) and `_lym/plan/09-origin-judge-core.md` (`TestReconcileOrigin_Snapshot`).
   - Both are tier1 tests "in package `fabricengine`" built "on a hubforge pair".
   - `internal/hubforge` imports `internal/fabricengine` in production (`internal/hubforge/seed.go:10`, `reanchor.go:11`, `recordsstop.go:5`). An internal test file of `fabricengine` that imports it is an import cycle and fails the card's build gate.
   - No current package-`fabricengine` test file imports hubforge (grep found none among 68 files).
   - Fix: place both tests in `fabricengine_test` and reach `pushClassified` and `runRule` through `export_test.go` accessors.

2. **MEDIUM · consistency** — `_lym/plan/12-board-test-moves.md`, `TestPush_BoardReplay` and its fixtures.
   - The moved `pushclasses` rows call helpers private to `fabricengine_test`, which `boardengine_test` cannot see: `newFailureFixture`/`recordsFixture` (`boardBranch`, `originCommit`), `fakePushNotifier` (`options`), `requireOneFailedPush`, `requireNoFailedPush`, `boardEntry`, `boardFileOf`, `boardTitles`, `withBoardTitle`, `bareBranchTip`.
   - Evidence: `internal/fabricengine/pushclasses_integration_test.go:45-81` and the helper calls of the row `ABoltPushRejectedOverAnUnrelatedOriginEntryIsReplayedAndLands`.
   - The card lists only `reconcileHub`, `newReconcileHub`, `writeRunStatus`, `teardownRecorder` and `requireEntry`, so the moved tests do not compile from the card as written.

3. **MEDIUM · design** — `_lym/plan/04-origin-reads-seams.md`, `TestInstallRequiredSeams`' run-lock check.
   - It asserts that the run-lock adapter "names what shed takes" through a moving `ReconcileOrigin`.
   - Until card 11, `tryPrimeLock` still reads `shedrun.PrimeRunLock` directly (`internal/fabricengine/originreconcilemoves.go:45`). The row passes at card 4 without touching the adapter.
   - It checks the adapter only once card 11 lands, so this row belongs to card 11 or after it.

4. **MEDIUM · consistency** — `_lym/plan/04-origin-reads-seams.md`, `TestInstallRequiredSeams`' codec check.
   - The card says the transition is "planted as `TestOriginCodeTip` plants one". The planting helper `plantRunTransition` and `pairFixture` are `fabricengine_test`-private (`internal/fabricengine/codetip_integration_test.go:22-38`).
   - A test in `fabriccli_test` cannot reach them, so the card must state its own planting.

5. **LOW · consistency** — `_lym/plan/07-push-side-core.md`, `**Extends:** internal/fabricengine/doc.go`.
   - `internal/fabricengine/doc.go:651-652` lists the classes `outcomeClassOf` maps: "done, anchor stop, auth refused, rejected, destructive refusal, transport failure or failed".
   - Card 7 adds takeover, protected, replay_conflict, origin_ahead and deferred, so that sentence becomes false.
   - This is a rewrite of existing text, so the doc belongs under `**Edit:**` and not `**Extends:**`.

6. **LOW · consistency** — `_lym/plan/11-origin-moves-core.md`, `cmd/lym#uncontainedWriteAllowlist` (Uses).
   - The `originreconcilemoves.go` entry's reason reads "the path is shedrun's, never caller-derived" (`cmd/lym/uncontainedwrite_test.go:72-73`).
   - Card 11 makes `tryPrimeLock`'s `MkdirAll` path come from the process-installed `RunLocks` seam, which a test sets to its own path. The reason text becomes false, and the card claims the entry "stays true".
   - Fix: move the allowlist to `**Edit:**` and restate its reason.

**Tool calls**
1. `builder_test.go` reflection, plus a grep for removed symbol names in docs and scans: to check reflective seam tests and docs that name the symbols the cards delete.
2. `builder_test.go` setter list and `fabricengine/doc.go` grep: to check `TestBuilder`'s setter coverage and which doc lines describe the changed behaviour.
3. `fabricengine/doc.go` reconcile and push-classification sections, plus the `fabriccli` and `hubforge` doc greps: to read the doc text cards 7 and 9 to 11 could falsify.
4. hubforge's production import of `fabricengine`: to confirm the import cycle behind finding 1.
