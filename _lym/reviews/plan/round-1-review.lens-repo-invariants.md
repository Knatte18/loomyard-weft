```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: reflective seam tests and tier tags, doc text a card makes false, and whether the card can pass its own gate after the cards before it.

**Per-card list**
- Card 1: seams held, no new tests. Docs held: it edits `PATTERN.md`, `pattern/PATTERN-lym-files.md`, `lymdirs` doc. Order held.
- Card 2: seams held, since both new `Status` fields are `omitempty`. Tiers held: tier0 in existing files. Docs held: `valid` comment and `shedengine/doc.go` are listed. Order held.
- Card 3: tiers held. `testmain_test.go` is an untagged support file and `derive_test.go` is tier0. Docs held. Order held: it uses card 2's `StateCancelled`.
- Card 4: tiers held (tier0). Docs held. Order held.
- Card 5: tiers held. `read_test.go` is tier1 on `hubforge`. Docs held. Order held.
- Card 6: seams held, since `RunView` has no seam marker. Tiers held. Docs held: `dobbyengine/doc.go` is listed. Order held: it needs cards 3–5, which come first.
- Card 7: tiers held. Order borderline: the nil behaviour of the new `startGuard.Act` seam is not stated (finding 6).
- Card 8: tiers held (tier0). Order held: a nil `actOf` reads as act.
- Card 9: tiers held. `intent_test.go` is tier0, and `PrimeName` reads through `dotgit` in-process, so no git process starts. **Docs broken** (findings 2, 3). **Order broken** (findings 1, 4).
- Card 10: **order broken** (finding 1). **Its Intent is contradicted** (finding 4).
- Card 11: tiers held. The tier0 `clearDoneRun` table takes fakes. Docs: one LOW (finding 7).
- Card 12: seams held. `landingshed#PullRequestDeps` is built only in `loomcli/landingdeps.go:347` and `envkit/envkit.go:224`, and the card covers both. Tiers held: `prrecord_integration_test.go` is tier1. **Docs: one LOW** (finding 5).
- Card 13: seams held. `landingshed#Deps` is built only in `loomcli/landingdeps.go:101` and `envkit/envkit.go:184`, and the card fills both. Docs held. Order held.
- Card 14: tiers held. New tests sit in tier1 `cli_test.go` and tier0 `intent_test.go`, matching their files. Order borderline (finding 6).

**Findings**

1. **MEDIUM, consistency.** Card 10 (cards 9 and 10 together): no card lists `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`. Its subtest `CancelMarkerRefusesTheStep` calls `takeCancelMarker`, which lives in `cancelmarker.go`. When card 10 deletes that file, battencli's test binary stops compiling. Evidence: `internal/battencli/startguard_integration_test.go:103-118`.

2. **MEDIUM, consistency.** Card 9: the battencli package doc in `internal/battencli/cli.go:19-30` describes the old cancel order. That order removes the records, takes the cancel marker, and "remove[s] the marker last". The `preDrive` field comment at `cli.go:282` also says it "re-reads the cancel marker". Card 9's Intent changes all of this, and neither card 9 nor card 10 lists `cli.go`. Card 10's own rule to leave no "cancel marker" in package docs then fails.

3. **MEDIUM, consistency.** Card 9: the `Long` help of `battenCLI.cancelCmd` says the cancel "removes the run's records and scratch". With origin unreachable, it says the cancel "leaves its marker". It also lists the envelope keys. Card 9 keeps the record without `--remove`, drops the marker and adds `intent_pushed` and `record_kept`, but `cancelCmd` is not a target. Evidence: `internal/battencli/cancelcmd.go:54-76`.

4. **MEDIUM, consistency.** Cards 9 and 10: `TestCancelIntegration_BlockedRunWithNoPair` asserts `records_removed == true` for a cancel without `--remove` (`internal/battencli/cancel_integration_test.go:121,162`). Card 9 makes `RecordsRemoved` stay false without `--remove`, but card 9 does not list this test. Card 10 says the test "loses only" its marker-file check. Card 9's tier1 batch gate therefore fails, and card 10's Intent understates the edit.

5. **MEDIUM, consistency.** Card 9's targets miss test helpers that set fields it renames or retypes:
   - `cancelFake.deps` (`cancel_test.go:141`) sets `ReadMarker` with type `cancelMarkerReading`.
   - `newPreDriveReceiver` (`predrive_test.go:29`) sets `startGuard.Marker` with `cancelMarkerReading`.
   - `fakeSubstrates.teardown` (`pairteardown/teardown_test.go:117`) sets `cancelMarked`, which card 9 renames to `cancelling`.

   The card's own gate cannot compile unless the implementer edits symbols the card does not list.

6. **LOW, consistency.** Card 9: it marks `internal/pairteardown/doc.go` as `Extends`, but `doc.go:154` says the dobby-gap "counts only while … no cancel marker of the slug stands". That sentence has to be rewritten, which makes it an `Edit` target. Card 10 does not list `pairteardown/doc.go` either.

7. **LOW, consistency.** Card 12: the doc comment of `shedrun#PullRequestRecordLock` says the lock "sits beside that record under ScratchDir". Once the record moves to `RunDir`, that is false, and the function is not listed. Evidence: `internal/shedrun/paths.go:235-239`.

8. **LOW, design.** Cards 7 and 14 add seams without saying what a nil seam does. Card 7 adds `startGuard.Act`; card 14 adds `tenderengine.Deps.RunOwner` and `.PauseIntent`. Every other `tenderengine.Deps` func field is documented "nil reads/spawns none" (`internal/tenderengine/report.go:79-122`). Card 8, by contrast, says a nil `actOf` reads as act. Existing hand-built receivers and `Deps` literals in unlisted tests would otherwise call a nil func.

9. **LOW, consistency.** Card 11: `judgeStatus`'s skip reason says `lym batten cancel` "ends it, and a later cleanup removes it" (`internal/battencli/cleanup.go:542-555`). After cards 9 and 11, a cancel without `--remove` keeps the record, and cleanup refuses it while the pair stands. The message is now wrong for that case, and `judgeStatus` is not listed.

Tool calls:
1. `lym:seam` markers, test-file tiers, `TestClearDoneRun` tier and `fabricengine.List`: to check seam tests and tier0 purity, which the context did not show.
2. Doc mentions of the PR record and attempts, and constructors of landing deps: to check doc falsification and seam fill sites.
3. `cancelcmd.go` help text and envelope-key assertions: to check help and test invariants that card 9 changes.
4. Enclosing functions of the `records_removed` assertions in `cancel_integration_test.go`: to attribute them to tests.
