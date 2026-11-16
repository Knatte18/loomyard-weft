# Card 7 — read-run-owed-and-tip

**Create:**
- `plan:internal/fabricengine#CodeTipReading` -> `type CodeTipReading struct`
- `plan:internal/fabricengine#OriginCodeTip` -> `func OriginCodeTip(prime *lymcwd.Location, slug string) (CodeTipReading, error)`
- `plan:internal/fabricengine#FailedPushesOwedBy` -> `func FailedPushesOwedBy(hubPath, slug string) ([]FailedPush, error)`
- `plan:internal/fabricengine#pushJob.resetUnnamedTip` -> `func (j *pushJob) resetUnnamedTip() (reset bool, err error)`
- `plan:internal/fabricengine_test#TestOriginCodeTip` -> `func TestOriginCodeTip(t *testing.T)`

**Edit:**
- `internal/fabricengine#pushJob.repushOverOriginContained`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/pairteardown#runReader`
- `internal/pairteardown#hubRunReader`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#runFakes`
- `internal/pairteardown#runFakes.reader`
- `internal/pairteardown#TestReadRun`
- `internal/pairteardown/doc.go`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`

**Uses:**
- `plan:internal/fabricengine#lastTransitionCodeTip`
- `internal/fabricengine#ListFailedPushes`
- `internal/fabricengine_test#newPairFixtureAt`

**Intent:**
`ReadRun` fills the two inputs nothing fills today (D1, D2).

The owed push: `FailedPushesOwedBy` returns the failed-push records of class `transport` or `teardown_lock` whose `Run` is the slug (card 3), newest `LastSeen` first.
A record naming another run, or none, is not this run's: an owed push of the prime's records branch written by one run's transition never holds another run.
`runReader` gains `owedPush func(slug string) ([]fabricengine.FailedPush, error)`; `read` sets `Inputs.PushOwed` when one stands, `Inputs.PushOwedError` to the newest record's `Error` and `Inputs.PushOwedWorktree` to its `RepoPath`.
Any other class is no owed push: such a push is no transient failure and the run halted on it (card 8).

The code tip: `OriginCodeTip` reads, with no network call, from origin's fetched records branch of the slug, the code tip the run's last transition names (`lastTransitionCodeTip`),
whether origin's fetched code branch holds it, the machine of the child record's last history entry at that tip, and that commit's time.
`CodeTipReading` holds `Tip`, `Missing`, `Machine` and `At`; a slug with no records branch on origin reads not missing.
`runReader` gains `codeTip func(slug string) (fabricengine.CodeTipReading, error)`; `read` sets `CodeTipMissing`, `CodeTip` and `CodeTipMachine` from it.
Until the tip lands the act rule answers `wait` (D2).

The owner's guard (D2): `repushOverOriginContained`, on finding origin's code branch ahead, calls `resetUnnamedTip` for a task pair.
It fetches the records branch and reads the tip the run's last transition on origin names (`lastTransitionCodeTip`), the record's operand.
When origin's code tip is not that tip and not an ancestor of it, it resets origin's code branch with a lease at origin's tip to that named tip and pushes again.
History is not searched: a tip given up through `ReturnToPreviousTransition` (card 12) stays named by an older commit's `Warp-SHA`, so only the last transition counts.
So a tip left by an owing machine that died before its re-read goes too; origin's tip equal to the last transition's, or behind it, keeps the rejection as today.

Tests:
- `TestOriginCodeTip` (tier1): rows for a named tip origin's code branch holds, one it lacks (with the machine and time read), and a slug with no records branch.
- `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` (tier1) gains the owner's guard rows:
  origin's code branch carries a commit past the tip the last transition names, and the owner's push resets it under the lease and lands its own code side;
  origin's code tip equal to the tip the last transition names keeps the rejection, with origin unchanged.
- `TestFailedPushRecord` (tier0) gains rows for `FailedPushesOwedBy`: a pair's transport record and a prime records record whose `Run` is the slug count; a `not_fast_forward` record does not; a prime records record naming another run, or no run, does not.
- `TestReadRun` (tier0) gains rows through `runFakes`' two new fields: an owed push sets `PushOwed` with its error and worktree; a missing tip sets the three code-tip inputs; a read error of either names the read and the slug.

**ImpactSummary:** Fills two act-rule inputs every `ReadRun` caller already judges, so cancel, resume and batten's own verbs now meet the owed-push and code-tip waits their `Aside` does not drop; the code-side guard acts only on a rejected task-branch push.
