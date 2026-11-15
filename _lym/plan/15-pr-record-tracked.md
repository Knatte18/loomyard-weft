# Card 15 — pr-record-tracked

**Create:**
- `plan:internal/shedrun#PullRequestRecordRel` -> `func PullRequestRecordRel(l *lymcwd.Location, runID string) string`
- `plan:internal/shedrun#LegacyPullRequestRecord` -> `func LegacyPullRequestRecord(l *lymcwd.Location, runID string) string`
- `plan:internal/loomcli#TestPullRequestDeps_CommitsTheRecord` -> `func TestPullRequestDeps_CommitsTheRecord(t *testing.T)`

**Edit:**
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`
- `internal/shedrun#TestPathConstructors`
- `internal/pullrequest#Record`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.checkWiring`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#NewTestDeps`
- `internal/landingshed#PRReview.recordPullRequest`
- `internal/landingshed#PRReview.updateRecord`
- `internal/landingshed#PRReview.closeRecordedPullRequest`
- `internal/landingshed#PRReview.openPullRequest`
- `internal/landingshed#Finalize.dropRecordedPullRequest`
- `internal/landingshed#Finalize.openAtOnce`
- `internal/landingshed#TestPRReview_Call`
- `internal/landingshed#TestFinalize_PullRequest`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#pawlRetiredMove`
- `internal/loomcli#TestStatusCommitPathspec`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomcli/doc.go`
- `internal/pullrequest/doc.go`
- `internal/landingshed/doc.go`
- `internal/shedrun/doc.go`

**Intent:** The pull-request record moves whole from the run's scratch into the pair's tracked records, and every write of it commits and pushes at once (D3).
- `PullRequestRecord` returns `pull-request.json` under `RunDir(l, runID)`; `PullRequestRecordLock` takes its path from `lock.TrackedFileLockPath`, as `PATTERN-lym-files` puts a tracked file's lock;
  `PullRequestRecordRel` is its anchor-relative pathspec; `LegacyPullRequestRecord` is the old scratch path, read only for the one-time move below.
- `pullrequest.Record`'s doc says the record is tracked with the run's records; its session ids hold only on the machine that wrote them, and a session whose transcript is absent starts fresh (D3).
- `PullRequestDeps` gains `CommitRecord func() error`, a required field: `PRReview.checkWiring` names it when it is unset, as the row's contract says of every field but the beam ones.
  Every successful write or removal in the listed `PRReview` and `Finalize` members calls it and returns its error.
  `newTestDeps` and `NewTestDeps` set a recording fake, so the existing PR-Review and Finalize tests pass the wiring check and count the calls.
- `pullRequestDepsGuardedBy` fills `CommitRecord`: it commits `PullRequestRecordRel` when the file is on disk or the records HEAD tracks it, then spawns the pair's records push.
  It also moves a record found only at `LegacyPullRequestRecord` to the new path once, so a run in flight across the change keeps its pull request.
- `statusCommitPathspec` stages the record whenever it is on disk, so a transition carries it too.
- `prContinueCmd` commits after `prContinueVerb` updates the record, and `pawlRetiredMove`'s scratch removal commits the record's removal, each through the same commit.

Tests:
- `TestPathConstructors` (tier0) asserts the record and its rel under the run directory and its lock outside `_lym`.
- `TestPRReview_Call` and `TestFinalize_PullRequest` (tier0) gain rows that count `CommitRecord` calls per write and return its error.
- `TestStatusCommitPathspec` (tier0) gains a row with the record on disk.
- `TestPullRequestDeps_CommitsTheRecord` is tier1, over a `hubforge` pair: a write through the deps leaves the record committed on the records side, and a legacy scratch record is moved and committed; that is the record a pull on another machine reads.

**ImpactSummary:** The record path moves under the tracked run directory; landingshed's six record writers gain a commit call, and loomcli's three record writers and the transition pathspec follow.
