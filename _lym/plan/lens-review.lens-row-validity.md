```yaml
lens: row-validity
outcome: reported
```

Lens row-validity: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

1. BLOCKING — Card 15 (Darn-Write record refusal), `TestDarnWrite_Call`: the new pre-spawn check stats `RecordPath` on disk, but the darn fixture is built over fake paths. `newDarnFixture` (internal/loomshed/darnwrite_test.go:25-33) uses `/work/description.md` and has no temp dir and no record file. Once the stat is added, every existing row ("first spawn tells nothing and commits", "prior work…", "gate-failed Stuck…") halts on "missing record" before the session runs, so they fail. The card does not list `internal/loomshed#newDarnFixture` or `internal/loomshed#darnFixture`, and does not say the fixture needs a temp dir with a record. The row "a stop marker written by the fake session" also needs `darnSession.Call` (darnwrite_test.go:41-44) to write the marker file. That helper change is unlisted too.

2. MEDIUM — Cards 13 and 15, the new `DarnDeps` seams: `CheckRecord`, `CommittedRecord` and the two paths have no stated nil or zero-value meaning. A hand-built `newDarnFixture` leaves them nil. Card 15 adds `CommittedRecord` to `darnWriteEntry`'s required seams, but neither card adds the seams to `darnWrite.checkSeams` or says whether a nil `CheckRecord` means "no guard" or a named error. Today's `darnWrite.checkSeams` names each seam it requires (internal/loomshed/darnwrite.go). The rows "a clean guard is Done" (card 13) and "with `CommittedRecord` true it names the restore" (card 15) depend on the fixture defaults, which neither card states.

3. BLOCKING — Cards 11, 13 and 15, the seam-reflection tests: wire leaves new Env seams nil (`RecordWrite.ReadBody` and `RecordWrite.Commit` in batten; `Darn.CheckRecord` and `Darn.CommittedRecord` for a tapestry or batten seed). `TestWire_EverySeamFilled` in both loomcli and battencli fails on any nil seam missing from its `intentionallyNil` list (internal/loomcli/wiring_seams_test.go:53-55,157; internal/battencli/wire_seams_test.go:122-124,185). Cards 13 and 15 list both tests only under `Uses`, and card 11 lists the battencli one under `Uses`. So the intentionallyNil entries the change forces are not planned: the `Darn.*` entries in both packages, and `RecordWrite.*` in battencli, which never fills it.

4. MEDIUM — Card 19 (`lym loom` refusals), `TestResolveReviewTarget`: the new `reviewTargetDeps.seedRecipe` seam has no stated nil meaning, and the test's fake builder `fakeReviewTarget` (internal/loomcli/review_test.go:37) is not listed on the card. Every existing subtest, and every other hand-built `reviewTargetDeps`, would call a nil `seedRecipe` and panic. The new batten-refused and pawl-resolves rows need `fakeReviewTarget` to answer a recipe per target.

5. LOW — Card 9, the new `TestResume_DoesNotRestartAtRowOne` row ("current row Preflight, history past Preflight, `start_sha` set, proceeds"): row 0 (Preflight) is always replaced with `fakeAlwaysDoneProducer` (internal/loomrecipe/resume_test.go:66), and once the Tapestry-Preflight row is deleted nothing in the code under test can refuse. The row cannot fail on a wrong Preflight; it only pins the row's absence, which `wantProducerTable` already pins. The existing `shed1.Producers[1]` substitution (resume_test.go:67-68, also :283-284) must go or it replaces Discussion-Write; the card lists the tests but not this.

6. LOW — Card 9, forced edits: `TestResume_CrashRecoveryRecallsUnconditionally`'s doc says its second run blocks at Tapestry-Preflight (resume_test.go:115-121), and `sequence_test.go:45-48` describes row 2 as Tapestry-Preflight. Neither test is on card 9, so both texts go stale. `TestSequence_FullRunBlocksAtMergeIn`'s expected order may also name the row.

7. LOW — Card 3, test economy: the `TestBuild_Errors` row asserting the contract refusal's `shedbuild:` prefix repeats what `TestBuild_LoomContract`'s rows already assert through `Build`. The new test's valid-recipe row only asserts the error "is not a contract refusal", which passes on any other error as well.

8. LOW — Card 15, the stop-marker halt reason "the change description's text read from the run's description": `DarnDeps` gains no description-path field. The card does not say whether the text is read from the session's `pointer.Path` or elsewhere, so the row asserting "halts with the description text" has no stated source.

Calls made:
- internal/loomrecipe/resume_test.go:58-125 — to see how the resume fixture replaces Preflight, which the card 9 row relies on.
- internal/loomcli/decision_test.go `decisionFake`, and `reviewTargetDeps` / `TestResolveReviewTarget` — for the fake fields the card 16 and card 19 rows need.
- internal/loomshed/darnwrite_test.go:1-120 — for the darn fixture and fake session the card 13 and card 15 rows run on.
- `intentionallyNil` and `nilLegal` lists in loomcli, battencli and envkit — to check how the seam-filled tests treat new nil seams.
- internal/loomrecipe/resume_test.go:126-168 and grep for Tapestry-Preflight in loomrecipe tests — for tests depending on the removed row.
- internal/loomcli/review.go:21-40 and review_test.go:32-70 — for how `reviewTargetDeps` is built in tests.
