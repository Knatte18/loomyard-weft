# Card 11 — pull-request-record-tracked

**Create:**
- `plan:internal/shedrun#PullRequestRecordRel` -> `func PullRequestRecordRel(l *lymcwd.Location, runID string) string`
- `plan:internal/loomcli#TestPullRequestRecord_ReachesAnotherCloneThroughOrigin` -> `func TestPullRequestRecord_ReachesAnotherCloneThroughOrigin(t *testing.T)`

**Edit:**
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestLocksAreDistinct`
- `internal/pullrequest#Record`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.recordPullRequest`
- `internal/landingshed#PRReview.updateRecord`
- `internal/landingshed#PRReview.closeRecordedPullRequest`
- `internal/landingshed#PRReview.resumeSession`
- `internal/landingshed#Finalize.openAtOnce`
- `internal/landingshed#Finalize.dropRecordedPullRequest`
- `internal/landingshed#prReviewTestDeps`
- `internal/landingshed#TestPRReview_Call`
- `internal/landingshed#TestPRReview_Session`
- `internal/landingshed#TestFinalize_PullRequest`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#prContinueVerb` -> `func prContinueVerb(out io.Writer, recordPath, lockPath string, publish func() error) int`
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#TestPRContinueVerb`
- `internal/loomcli#pawlRetiredMove`
- `internal/loomcli#TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord`
- `internal/loomcli#TestPullRequestDeps_RecordChangesWatchesTheRunsPullRequestRecord`

**Extends:**
- `internal/shedrun/doc.go`
- `internal/pullrequest/doc.go`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/shedrun#RunDir`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#StatusPushLockWait`
- `internal/pairteardown#Teardown.closePairPullRequest`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Intent:**
The pull-request record of `internal/pullrequest/record.go` moves whole into the pair's tracked records, and a write of it commits and pushes at once (D3).
`shedrun.PullRequestRecord` returns `pull-request.json` under `shedrun.RunDir`; its lock moves to the checkout's lock directory through `lock.TrackedFileLockPath` (PATTERN-lym-files); `PullRequestRecordRel` is its anchor-relative form for a commit pathspec.
`landingshed.PullRequestDeps` gains `Publish func() error`, called after every write, update and removal of the record: `PRReview.recordPullRequest`, `PRReview.updateRecord`, `PRReview.closeRecordedPullRequest`, `Finalize.openAtOnce` and `Finalize.dropRecordedPullRequest`; a publish error fails the row with its cause and the re-run as way forward.
`pullRequestDepsGuardedBy` sets it: commit `PullRequestRecordRel` (or its removal) with `fabricengine.CommitAnchoredPaths`, then push the pair with `fabricengine.PushPairAnchored`, committing only under the push skip, which pushes nothing.
`statusCommitPathspec` stages the record when it exists, so a transition's commit carries it too.
`prContinueVerb` takes the same publish and calls it after its update; `loomCLI.prContinueCmd` passes it.
`pawlRetiredMove` reads and removes the record at the new path.
`pairteardown`'s read of the record goes through `shedrun.PullRequestRecord` and needs no change.
The record's session id holds only on the machine that wrote it (D3): `PRReview.resumeSession` starts the PR-Review session fresh when the engine's resume of the recorded session fails for a transcript absent here, through the resume check the shuttle provider seam already owns (PATTERN-shuttle-provider-seam); grep it first and add only the fall-back that is missing.
The docs say where the record lives and that a write publishes it; `pullrequest.Record`'s doc no longer says scratch directory.
Grep for the old `pull-request.json` scratch path in tests and move each to the new path.
Tests: `TestPathConstructors` and `TestLocksAreDistinct` (tier0) pin the new path and lock; the PR-Review and Finalize tests assert `Publish` after each write;
`TestPullRequestRecord_ReachesAnotherCloneThroughOrigin` (tier1) writes a record through the wired seams in one clone of a `hubforge` hub and reads it in another after a pull.

**ImpactSummary:** the pull-request record moves from `.lym` to the pair's `_lym` records and every write of it is committed and pushed; `prContinueVerb` takes the publish.

**Commit:** 11: The pull-request record lives in the pair's records and is pushed on every write
