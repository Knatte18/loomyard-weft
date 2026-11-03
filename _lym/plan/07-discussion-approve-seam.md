# Card 7 — discussion-approve-seam

**Create:**
- `plan:internal/loomcli#storeDecisionRecord` -> `func storeDecisionRecord(location *lymcwd.Location, slug string, record []byte) error`
- `plan:internal/loomcli#approveDiscussion` -> `func approveDiscussion(location *lymcwd.Location, runID string) error`
- `plan:internal/loomcli#TestApproveDiscussion_Real` -> `func TestApproveDiscussion_Real(t *testing.T)`
- `plan:internal/loomrecipe#discussionApproveSeamFault` -> `func discussionApproveSeamFault(parsed shedbuild.Recipe) string`
- `plan:internal/loomrecipe#TestDiscussionApprovalRowsCarryApproveSeam` -> `func TestDiscussionApprovalRowsCarryApproveSeam(t *testing.T)`

**Edit:**
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#TestBouncerEntry_ApproveSeam`
- `internal/shedrecipe/entries_gate.go`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#intentionallyNil`
- `internal/loomrecipe#TestShippedRecipe_SeamsWiredOnPlanBouncerOnly`
- `internal/loomrecipe#TestApproveSeam_FailsToBuild`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/shedrecipe/doc.go`
- `plan:internal/boardengine#Board.StoreDecisionRecord`
- `plan:internal/boardengine#Board.DecisionRecord`
- `plan:internal/boardengine#ErrRecordEntryDone`
- `plan:internal/loomrecipe#discussionApprovalRow`
- `internal/loomrecipe#approveSeamFixture`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe/doc.go`
- `internal/loomcli#realSeamFixture`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/shedadapters#TestBouncer_ConvergedSettle_CallsApproveThenCommit`
- `internal/shedadapters#TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit`
- `internal/shedadapters#TestBouncer_Skip_TrueSpawnsNothingAndApprovesBeforeCommit`
- `internal/shedadapters#TestBouncer_Skip_FailingApproveSkipsCommit`
- `internal/shedadapters#TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling`
- `internal/shedadapters#TestBouncer_Circling_AcceptWithFailedCommitResumesByClearingWithoutApprove`

**Intent:**
Store the decision record on the board entry when Discussion-Review approves (D6), and guard the row that does it (D7).

**Recipe.**
The Discussion-Bouncer row in `tapestry-recipe.yaml` gains `approve_seam: discussion`, with a comment: the seam copies the approved record onto the run's board entry, and runs before `commit_seam`.
The whole Plan-Bouncer comment block on `approve_seam: plan` is reworded: this row is the only one with `approve_seam: plan`, the one row that knows the plan passed review; Discussion-Bouncer carries `approve_seam: discussion` instead; and Code-Bouncer gets no `approve_seam`, since its artifact carries no approval flag and an absent key leaves the seam nil as an omitted `commit_seam` does.
The sentence that neither Discussion-Bouncer nor Code-Bouncer gets the key goes.
No row is added, so the row list, the entry numbers and the rows `cartoon` copies stay as they are (`PATTERN-shed-recipe-registry`).

**Env and entry.**
`Env` gains `ApproveDiscussion func() error`, documented as the closure that copies the run's decision record onto its board entry, read by the Bouncer entry's `approve_seam: discussion`.
`bouncerEntry`'s `approve_seam` switch accepts `discussion` beside `plan`, guarded by `requireSeam("Bouncer", "ApproveDiscussion", env.ApproveDiscussion)`.
Its unknown-value error names both accepted values.
Its doc comment, the switch comment, and the comment above the `approve_seam` read that says the key resolves to `env.ApprovePlan` and nothing else, which today each name `env.ApprovePlan` alone, name both, and so does the file comment of `entries_gate.go`, which lists the Env closures `approve_seam` resolves against.
`envkit.FullEnv` fills `ApproveDiscussion` with a no-op, and battencli's `intentionallyNil` gains `"ApproveDiscussion": "loom-only seam, batten has no discussion segment"`.

**Wiring.**
`storeDecisionRecord(location, slug, record)` opens the hub board with `boardengine.OpenHub(location.HubPath)` and returns `StoreDecisionRecord(slug, record)`'s error.
It is the one store call of loomcli, shared with cards 8 and 9.
`approveDiscussion(location, runID)` is the seam's body: it reads `loomengine.DiscussionDecisionRecord(location)`, the working copy the approval commit then lands, and calls `storeDecisionRecord` for `shedrun.ResolveRunID(location, runID)`, the slug `SetRunStatus` is written for.
- A read error, or a store error other than a done entry, is returned, so the settle fails.
- A store refused with `ErrRecordEntryDone` is logged at Warn and returns nil, so the settle goes on, as the decision record's `Added after Discussion` entry "Store failures at approval: only a done entry warns" rules: a landed entry keeps the record it last stored (D2), and no resume can change that.
  Every other store failure, an absent entry and a board `OpenHub` cannot open included, fails the settle, since a resume can store again once the fault is fixed.
`loomCLI.wire` fills `ApproveDiscussion` with `func() error { return approveDiscussion(location, c.runID) }`.

The Bouncer already calls its approve seam on every approving path, after `writeCarryOver` and before the commit seam: a judged CONVERGED settle, an accepted round, a decision accept, a skipped review and a zero-round settle.
`internal/shedadapters` therefore needs no change; its tests in `Uses` pin, per path, that the seam runs in that order and that a failing approve fails the settle, skipping the commit, and the new real test below pins what the seam stores.
A failed store halts the run at Discussion-Bouncer with the error.
On a converged, skipped or zero-round settle a resume re-settles and stores again, since the store is idempotent.
On an accepted round the accept is settled before the seam runs, so a resume archives the settled generation and re-enters Discussion-Review without a second approve, as it already does for a failed `ApprovePlan` (`TestBouncer_Circling_AcceptWithFailedCommitResumesByClearingWithoutApprove`); the record is stored at the segment's next approval, as the decision record's `Added after Discussion` entry "A resume after an accepted round does not store again" rules.
Until then `lym loom decision store` refuses such a run, since its row is inside Discussion-Review and so not past the approval as card 6 judges it.
`loomcli/doc.go` gains one sentence on the approve seam's store.

Bound: a `lym loom goto` past Discussion-Bouncer skips this store. `decision add` (card 8) and `lym loom decision store` (card 9) store such a run's record.

**Tests.**
- `TestBouncerEntry_ApproveSeam` gains two subtests. `DiscussionResolvesToApproveDiscussion` asserts one `ApproveDiscussion` call and no `ApprovePlan` call on an approved settle. `NilApproveDiscussionRefused` asserts the error names `ApproveDiscussion`.
- `TestApproveSeam_FailsToBuild`'s unrecognized-value row uses `webster`, since `discussion` is now accepted, and its doc comment names `plan` and `discussion` as the accepted values, any other value failing to build.
- `TestShippedRecipe_SeamsWiredOnPlanBouncerOnly`'s `approve_seam` map gains `Discussion-Bouncer: discussion`, and its doc comment says so.
- `TestSequence_FullRunBlocksAtMergeIn` sets `env.ApproveDiscussion` to a counter and asserts it ran exactly once over the run.
- `TestDiscussionApprovalRowsCarryApproveSeam` (tier 0) checks the guard. `discussionApproveSeamFault(parsed)` returns the first fault, or "". A fault is a discussion approval row, as `discussionApprovalRow` finds it, without `approve_seam: discussion`, or any other Bouncer row carrying `approve_seam: discussion`. The test asserts no fault over every embedded recipe `IsLoom` accepts. It asserts a fault for a fixture whose DiscussionWrite row hands to a Bouncer lacking the key, and for a fixture whose Code-Bouncer carries it.
- `TestApproveDiscussion_Real` (tier 1) sets `BOARD_SKIP_GIT`, builds a hub pair as `realSeamFixture` does, upserts a board entry under the pair's slug through `boardengine.OpenHub`, and writes a record at `loomengine.DiscussionDecisionRecord(location)`. Its rows:
  - `approveDiscussion(location, shedrun.SelfRunID)` leaves `DecisionRecord` returning the working copy's bytes and the entry `Discussed`;
  - with the entry marked done, it returns nil, so the settle passes with its Warn line, and writes no record file;
  - with no entry under the slug, it returns the error naming `lym board upsert`;
  - with no record working copy, it returns the read error.

**ImpactSummary:** One recipe key, one Env seam and one Bouncer switch value. Every `shedrecipe.Env` builder is touched: `wire()`, envkit's `FullEnv`, and battencli's nil list. Tapestry is the only recipe with a DiscussionWrite row, so no other recipe changes behaviour.
