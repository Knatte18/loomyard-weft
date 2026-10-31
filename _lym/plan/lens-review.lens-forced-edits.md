```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits — what each card's change forces that no card lists.

**BLOCKING**

1. **Card 9 misses the sequence fixture.** It deletes the Tapestry-Preflight row, but `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn` is not on the card. Its fixture and comments describe a status whose `current_producer` is `"Tapestry-Preflight"` and say "the real row 2 (Tapestry-Preflight) passes against this fixture".
   - Evidence: `internal/loomrecipe/sequence_test.go:45` and `:48`.
   - After the card the recipe has no such row, so the test either breaks or keeps describing a row that is gone.
   - Fix: list the test on card 9 and rewrite its fixture onto Preflight → Discussion-Write.

**MEDIUM**

2. **Card 9 leaves the orch role stencil stale.** `contracts/stencils/orch/orch-template-role.md:167` names `Tapestry-Preflight` as a pre-record row. It also says a `Decision for <slug>:` board note "counts as a parent decision, which … the darn writer read[s]".
   - D6 (card 15) makes a note never settle a darn stop.
   - Card 16 changes the no-record refusal so it names `resume`/`goto --to Record-Write` instead of a board note.
   - No card lists this stencil.

3. **Card 5 leaves `docs/overview.md` wrong.** `docs/overview.md:597` says the four driver and tackler keys of `loom.yaml` and `darn.yaml` override `shed.yaml`. Card 5 retires them from both modules but does not list `docs/overview.md`; cards 6, 9 and others list it for other paragraphs only.

4. **Card 12 misses the PR-Review snapshot path.** `internal/loomcli#pullRequestDeps` still sets `BoardEntryPath: loomengine.DiscussionBoardEntry(l)` (`internal/loomcli/landingdeps.go:289`). The PR-Review session lists that path in its prompt when it exists (`internal/landingshed/prsession.go:158`, `deps.go:364`).
   - Once darn stops writing `board-entry.md`, the code is harmless, but nothing records that only pawl now feeds it.
   - Card 12 lists neither `pullRequestDeps` nor `landingshed/deps.go`'s field doc.

5. **Card 12 misses the snapshot commit test.** It deletes `commitDarnDiscussion`, but `internal/loomcli/wiring_darn_integration_test.go`'s `TestCommitDarnDiscussion_Real` (line 25) asserts that `board-entry.md` is committed (line 68). Card 12's Delete lists `internal/loomcli#TestCommitDarnDiscussion_Real`, so that one is covered. However, `internal/landingshed#TestPRReview_Session` (`prreview_session_test.go:183`) still builds a `board-entry.md` answer key. Its scope after card 12 is pawl only, and no card says whether that test keeps its meaning.

6. **Card 15 may break a stencil marker test.** It drops the `support_log_path` marker and adds `stop_marker_path` to `darn-template-write.md`. `contracts/stencils#TestStencils_TopLevelMarkers` and `#TestStencils_MarkerPositions` (`contracts/stencils/markers_test.go:20,99`) pin stencil marker sets and positions, and card 15 lists neither; only card 17 lists `TestStencils_TopLevelMarkers`. The darn writer's row in that table was not confirmed (no direct grep hit), so verify before keeping or dropping this finding.

**LOW**

7. **Card 9's Preflight batcher check has no logging test.** `internal/loomshed#TestBatcherRows_StuckSurfacesTheBatcherError` carries a `//testtiming:keep` reason saying "each batcher-backed row logs its name and the batcher error text", and its table now holds Webster alone (`gatefindings_test.go:155-170`). Card 9 makes Preflight batcher-backed but does not say Preflight logs the batcher error, as Tapestry-Preflight did, and adds no row there.

8. **Card 9 leaves stale docs that name the removed row or check.**
   - `internal/shedrun/doc.go:6` names `CheckSeed`.
   - `internal/hubreconcile/doc.go:85` says "Tapestry-Preflight's retired-key refusal backs that up". That backstop is now Preflight's.
   - `contracts/specs/shed-recipe-spec.md:21` names `Tapestry-Preflight`. Card 3 edits this file earlier but does not cover this line.
   - `internal/loomcli/smoke_test.go:214` has a comment naming the row.

   None is listed on card 9.

9. **Card 6 leaves the implementer spec stale.** It moves `implementer` to `tapestry.yaml`, but `contracts/specs/implementer-spec.md:10` still says the row's engine is the one "`loom.yaml`'s `implementer` key names". Card 6 does not list that spec.

10. **Card 8 does not mention a historical tool.** `tools/tapestryrename/tables.go` and its test name `Tapestry-Preflight`/`Loom-Preflight`. The tool records a past rename and likely stays as is, but the card should say so.

Calls I made:
- One grep for callers of the removed builders, Tapestry-Preflight, RecipeOverride/ShedOverride, the darn Describe and the discussion validators, because the pre-read covered only a subset of call sites.
- One grep for the line context of the unlisted hits, plus the name of the darnengine test holding `ShedOverride`, to see whether they were already covered.
- A read of the conformance table, `TestBatcherRows`, orch role line 167, the gotoentry test name and the doc key references, to check coverage and stale docs.
- A read of the sequence fixture, the hubreconcile doc line, the stencil marker pins and the `board-entry.md` references, to confirm the fixture dependency and the snapshot readers.
- A read of the stencil marker test names and the board-entry users in tests and `landingdeps`, to check cards 12 and 15's forced edits.
- A read of `pullRequestDeps` and the landingshed `BoardEntryPath` use, to judge the PR-Review snapshot reader after card 12.
