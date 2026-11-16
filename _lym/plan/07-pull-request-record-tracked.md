# Card 7 — pull-request-record-tracked

**Create:**
- `plan:internal/shedrun#PullRequestRecordRel` -> `func PullRequestRecordRel(l *lymcwd.Location, runID string) string`
- `plan:internal/landingshed#PullRequestDeps.writeRecord` -> `func (pr PullRequestDeps) writeRecord(rec pullrequest.Record) error`
- `plan:internal/landingshed#PullRequestDeps.updateRecord` -> `func (pr PullRequestDeps) updateRecord(mutate func(rec *pullrequest.Record) error) error`
- `plan:internal/landingshed#PullRequestDeps.removeRecord` -> `func (pr PullRequestDeps) removeRecord() error`
- `plan:internal/loomcli#publishPullRequestRecord` -> `func publishPullRequestRecord(l *lymcwd.Location) error`
- `plan:internal/landingshed#TestPullRequestRecordWritesArePublished` -> `func TestPullRequestRecordWritesArePublished(t *testing.T)`
- `plan:internal/loomcli#TestPublishPullRequestRecord` -> `func TestPublishPullRequestRecord(t *testing.T)`

**Edit:**
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`
- `internal/shedrun#TestPathConstructors`
- `internal/pullrequest#Record`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.recordPullRequest`
- `internal/landingshed#PRReview.updateRecord`
- `internal/landingshed#PRReview.closeRecordedPullRequest`
- `internal/landingshed#PRReview.openPullRequest`
- `internal/landingshed#Finalize.dropRecordedPullRequest`
- `internal/landingshed#Finalize.openAtOnce`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#prContinueVerb` -> `func prContinueVerb(out io.Writer, recordPath, lockPath string, publish func() error) int`
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#TestPRContinueVerb`
- `internal/loomcli#pawlRetiredMove`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Uses:**
- `internal/shedrun/doc.go`
- `internal/pullrequest/doc.go`
- `internal/landingshed/doc.go`
- `internal/loomcli/doc.go`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#PushPairAnchored`
- `plan:internal/fabricengine#Topology.TaskRecordsFile`

**ImpactSummary:** the run's pull-request record moves from its scratch directory into its tracked run directory, and every write of it commits and pushes the pair's records.

**Intent:**
Move the pull-request record of `internal/pullrequest/record.go` whole into the pair's records, where a write of it commits and pushes at once (D3).

`PullRequestRecord` returns `pull-request.json` under `RunDir` in place of `ScratchDir`.
`PullRequestRecordLock` returns its lock through `lock.TrackedFileLockPath`, under the checkout's `.lym/locks/`, never beside the tracked file.
`PullRequestRecordRel` returns the anchor-relative path a records commit stages.
`TestPathConstructors` pins the three.
`Record`'s doc says the record is tracked in the run's records and that its `SessionID` and strand fields hold only on the machine that wrote them.

`PullRequestDeps` gains `PublishRecord func() error`, which commits the record and pushes the pair's records; a nil `PublishRecord` publishes nothing.
The three new methods write, update or remove the record through `pullrequest`'s functions and then call `PublishRecord`;
a publish error is returned beside the written record, and the row's next write publishes again.
Every write site in the PR-Review and Finalize rows goes through them.
`pullRequestDepsGuardedBy` sets `PublishRecord` to `publishPullRequestRecord` over the task location.
`publishPullRequestRecord` commits `PullRequestRecordRel` when the file exists or git tracks it, so a removal commits its deletion.
It then pushes through `fabricengine.PushPairAnchored` with `StatusPushLockWait`.
`prContinueVerb` publishes after its update, and `prContinueCmd` passes the same publisher.
`pawlRetiredMove`'s scratch removal of the record publishes the removal.

`TestPullRequestRecordWritesArePublished` (tier 0, fakes) asserts each write, update and removal calls `PublishRecord` once, after the file changed.
`TestPublishPullRequestRecord` (tier 1, hubforge) writes the record on a task pair and publishes it.
It then reads the record back from origin's records branch through `Topology.TaskRecordsFile` from the prime, as another machine reads it after a pull.
`TestPRContinueVerb` passes a recording publisher and asserts it ran.

**Commit:** 7: the pull-request record is tracked in the run's records and every write of it is pushed
