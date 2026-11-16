# Card 12 — pull-request-record-tracked

**Create:**
- `internal/loomcli/prrecord_integration_test.go`
- `plan:internal/shedrun#PullRequestRecordRel` -> `func PullRequestRecordRel(l *lymcwd.Location, runID string) string`
- `plan:internal/landingshed#PullRequestDeps.publishRecord` -> `func (pr PullRequestDeps) publishRecord() error`
- `plan:internal/loomcli#publishPullRequestRecord` -> `func publishPullRequestRecord(check prePushCheck, l *lymcwd.Location) func() error`
- `plan:internal/loomcli#TestPublishPullRequestRecord_CommitsAndPushesTheRecord` -> `func TestPublishPullRequestRecord_CommitsAndPushesTheRecord(t *testing.T)`

**Edit:**
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#TestPathConstructors`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.recordPullRequest`
- `internal/landingshed#PRReview.updateRecord`
- `internal/landingshed#PRReview.closeRecordedPullRequest`
- `internal/landingshed#PRReview.openPullRequest`
- `internal/landingshed#Finalize.openAtOnce`
- `internal/landingshed#Finalize.dropRecordedPullRequest`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#prContinueVerb`
- `internal/loomcli#pawlRetiredMove`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestPRContinueVerb`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/pullrequest#Record`
- `internal/pullrequest/doc.go`

**Extends:**
- `internal/shedrun/doc.go`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/loomcli/doc.go`

**Intent:** The run's pull-request record becomes a tracked record in the pair's records, and every write of it commits and pushes at once (D3).

`shedrun.PullRequestRecord` moves the file to `pull-request.json` under `RunDir`; its lock stays under `ScratchDir`.
`PullRequestRecordRel` is its anchor-relative pathspec.
`PullRequestDeps` gains `RecordWritten func() error`, which commits and pushes the record.
`publishRecord` calls it, and every landingshed member that writes, updates or removes the record calls `publishRecord` after the write succeeds;
a failed publish is that member's error, so the row halts as on a failed write.
`publishPullRequestRecord` fills the seam in `pullRequestDepsGuardedBy`:
it commits `PullRequestRecordRel` when the file is on disk or tracked (`fabricengine.PathTracked`), so a removal commits too, and does nothing for a path that is neither,
then pushes through `fabricengine.PushPairAnchored`, both through the pre-push check's guarded commit and push.
`prContinueVerb` and `pawlRetiredMove` publish after their writes the same way.
`statusCommitPathspec` adds `PullRequestRecordRel` on the same on-disk-or-tracked rule, so any records commit carries it too.
`envkit.LandingDeps` fills `RecordWritten` with a no-op, and `TestNilSeams_ReportsExactPaths` names the new field among the paths it expects when it lists them.
`pullrequest/doc.go` and `Record`'s doc comment say the record is tracked in the run's records, with its lock local; `shedrun/doc.go` and `landingshed/doc.go` gain the matching sentences.

Tests: `TestPathConstructors` (tier0) pins the new path and its rel;
`TestPublishPullRequestRecord_CommitsAndPushesTheRecord` goes in `prrecord_integration_test.go`, on a `hubforge` hub:
a record written in one pair is committed and pushed, then read through `fabricengine.RecordsBranchFile` from a second copy of the hub after its fetch, and a removal reaches origin the same way;
`prrecord_integration_test.go` is tier1;
`TestPRContinueVerb` (tier0) asserts the publish runs after the update.
The seam tests in `Uses` need no change: production wiring fills the new field.

**ImpactSummary:** Moves one record path from `.lym` to `_lym` and adds a publish after each of its writes; every reader takes the new path from `shedrun` unchanged.

**Commit:** 12: pull-request-record-tracked
