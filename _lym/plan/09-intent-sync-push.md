# Card 9 — intent-sync-push

**Create:**
- `plan:internal/tendercli#owedIntentPush` -> `func owedIntentPush(slug string, err error, stderr io.Writer) error`
- `plan:internal/tendercli#TestOwedIntentPush` -> `func TestOwedIntentPush(t *testing.T)`

**Edit:**
- `internal/tendercli#commitRunIntent`
- `internal/tendercli#runIntents`
- `internal/tendercli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `internal/tendercli/doc.go`
- `internal/battencli#cancelDeps`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#clearRunPhase`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestCancelRun`
- `internal/loomcli#pullRequestRecordCommitter`
- `internal/loomcli#TestPullRequestDeps_CommitsTheRecord`
- `internal/loomcli#TestPrePushCheckGuardedSeams`

**Delete:**
- `internal/loomcli#prePushCheck.guardedSpawn`
- `internal/loomcli#spawnStatusPush`

**Uses:**
- `internal/pairteardown#PushRun`
- `internal/fabricengine#IsPushOwed`
- `internal/battencli#judgeOrigin`
- `internal/loomcli/doc.go`

**Intent:**
The tracked intents and the pull-request record push synchronously (D1).

`lym tender pause <slug>` and `lym tender resume <slug>`: `commitRunIntent` pushes the prime's records side through `pairteardown.PushRun(prime, prime, slug, RunPush{LockWait: StatusPushLockWait})` in place of `SpawnStatusPush`, and hands the push's error to `owedIntentPush`.
`owedIntentPush` answers nil for no error; for an owed push (`IsPushOwed`) it writes one line to stderr naming the slug, the owed push and its error, and that the intent lands with the prime's records branch's next push or `lym fabric push` from the prime, and returns nil: the verb acts here at once and owes its push (D1).
Any other push error it returns, so the verb fails naming it.
With origin unreachable the pause entry is already written when `Record` runs, so the run's processes are held here at once.
The owed intent's commit lands with the prime's records branch's next push (overview, "out-of-part").
The `runIntents` field docs say the intent is pushed and waited for.

`lym batten cancel`: with origin unreachable, `judgeOrigin` answers end-only (`cancel.go:604`, `:609`).
The end-only path already writes and commits the intent before `EndForCancel` (`cancel.go:370-379`); with card 8, `commitCancelRecord` owes that commit's push, and the output names the re-run of `lym batten cancel <slug>` as today.
`cancelDeps.SpawnPush` becomes `PushRecords func() (owed bool, err error)`, which `hubCancelDeps` fills with the same `PushRun` over the prime, reading `IsPushOwed`.
`clearRunPhase` calls it after the lifts; an owed push makes the output name that same re-run.

The pull-request record: `pullRequestRecordCommitter` pushes through `PushRun(prime, l, slug, RunPush{LockWait: StatusPushLockWait, TeardownWait: StatusPushLockWait, InsidePair: true})` behind the pre-push check's `guardedPush`, in place of the spawn.
Its last spawning caller gone, `prePushCheck.guardedSpawn` and loomcli's `spawnStatusPush` go (card 8 already dropped `TestCommitStatusSeam_Real`'s swap).

Tests:
- `TestOwedIntentPush` (tier0): no error answers nil and writes nothing; a transport `*PushFailedError` answers nil and writes the line naming the slug and `lym fabric push`; a `not_fast_forward` failure is returned.
  The owed branch is driven here at tier0, since a real unreachable origin waits out the push backoff (`gitrepo.PushBackoffDelays`), which no test may.
- `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (tier1): a slug pause over a slug with a batten record (the test seeds one) leaves origin's prime records branch holding the pause intent once the verb returns, and its resume leaves origin holding the cleared record.
- `TestCancelRun` (tier0) through `cancelFake.deps`: its `SpawnPush` calls become `PushRecords`;
  an owed `PushRecords` row after the lifts asserts the output naming `lym batten cancel <slug>` as the re-run.
- `TestPullRequestDeps_CommitsTheRecord` (tier1): origin's records branch holds the record once the committer returns.
- `TestPrePushCheckGuardedSeams` (tier0) drops its spawn row.

**ImpactSummary:** Moves three intent writers and the pull-request record from a detached spawn to a synchronous push; the verbs' outputs change only when the push is owed, and two loomcli helpers lose their last caller.
