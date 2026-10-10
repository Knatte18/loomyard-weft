# Card 24 — recovery-count-and-retry

**Create:**
- `plan:internal/websterengine#ErrRecoveryExhausted` -> `var ErrRecoveryExhausted error`
- `plan:internal/websterengine#RecoveryRetry` -> `func RecoveryRetry(geom Geometry, st *State, n int) (retry bool, recoveries int)`
- `plan:internal/websterengine#nonMergeCommitsBetween` -> `func nonMergeCommitsBetween(worktree, base, head string) (bool, error)`
- `plan:internal/websterengine#realGit.NonMergeCommitsBetween` -> `func (realGit) NonMergeCommitsBetween(worktree, base, head string) (bool, error)`

**Edit:**
- `internal/websterengine#BatchState`
- `internal/websterengine#Git`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#existingReportRemedy` -> `func existingReportRemedy(number int, recorded *BatchState, retry bool) string`
- `internal/websterengine/gitfake_test.go`
- `internal/websterengine/gitwrap_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/doc.go`
- `//contracts/specs/refusal-spec.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Count a batch's recovery spawns, give a dead batch whose recovery committed one more recovery, and bound it (D12).
`BatchState` gains `Recoveries`, the count of the batch's counted recovery spawns, and `RecoveryStartSHA`, HEAD when the record's recovery was spawned.
`recoverSpawn` reads HEAD into `RecoveryStartSHA` before `Starter.Start`, and the record it returns after a successful start carries `Recoveries` as the prior's plus one, except a spawn whose prompt renders an amendment no earlier spawn rendered, which keeps the prior's count; a start that fails counts nothing, and `StartSHA` keeps inheriting the original bracket's start.
A HEAD read that fails returns its error before `Starter.Start`, so no strand starts and nothing is counted or recorded, with the way forward that it is transient and to re-run `lyx webster recover-batch NN`; a spawn never records an empty `RecoveryStartSHA`, which would silently disable the retry.
That one pre-`Start` read also supplies the `start` fallback for a prior without a `StartSHA`, and the post-`Start` `HeadSHA` call, which today can fail after the strand started and leave it unrecorded, is removed, so no HEAD read sits between `Starter.Start` and the returned record.
`BeginBatch` carries `Recoveries` and `RecoveryStartSHA` over on a re-begin beside `StartSHA`, `AuditWarnings` and `ForkTranscripts`; a rebaseline keeps batch records and so the count.
`RecoverSpawnOrAttach` refuses a counted spawn for any caller once `Recoveries` is 2, wrapping `ErrRecoveryExhausted` and naming `lyx webster reset --to batch-start --batch NN` then `recover-batch`, or `reset --to start`; below the cap it respawns over a terminal prior as today, and an amendment re-run at the cap spawns uncounted.
`RecoveryRetry` is the one function both verbs read: true for a terminal `dead` recovery-kind prior while `Recoveries` is below 2 when the dead recovery committed, that is HEAD descends from a non-empty `RecoveryStartSHA` and the first-parent range from it to HEAD holds a commit that is not a merge; an empty `RecoveryStartSHA`, a HEAD a reset moved off it and a range of merge-ins only all give false, and a git read that fails, a start absent from the store included, gives false and is logged at Warn.
`Git` gains `NonMergeCommitsBetween`, `realGit` implements it over `nonMergeCommitsBetween` in gitwrap.go through the checked `gitexec.Run`, and `gitfake_test.go`'s fake implements it over its recorded commits.
`existingReportRemedy` takes the retry verdict `BeginBatch` computes with `RecoveryRetry`: terminal dead with retry names `lyx webster recover-batch NN`, otherwise the run ends stuck naming the batch.
Bound: at most two counted recoveries per batch for every caller, plus one uncounted re-run per amendment; a record save that fails after a started strand leaves that spawn uncounted, so the cap then allows one spawn more.
`doc.go`'s cold-recovery section states the count, the retry rule and the cap; the refusal-spec row `report already present` (the terminal-dead remedy) is reworded, a new row `recovery exhausted` names the way forward, and a new row `recovery start unread` names the HEAD-read failure's re-run.
A new `TestRecoveryRetry` over the git fake pins the acceptance rows: a dead recovery with commits past its start is true with recoveries 1; a dead recovery without its own commits, over a fork that committed, a record with no start, a HEAD a reset moved off it, a range of merge-ins only, a dead recovery at two recoveries, and a terminal `stuck` and a terminal `failed` recovery prior each holding commits past its start are each false; a failed git read is false and its row asserts the captured Warn line.
The log capture swaps the logger's process-global output, so `TestRecoveryRetry` and its rows run without `t.Parallel` under a comment naming that output.
`TestRepositoryProbes` runs `nonMergeCommitsBetween` against its scratch repository, so the real-git side the fake stands in for is pinned too: a commit past the start is true, a first-parent range of merge-ins only is false, and a start SHA absent from the store is an error.
`TestRecoverSpawnOrAttach` gains the rows: the count rises on a spawn, a failed start leaves it unchanged, a failed HEAD read starts nothing and leaves the record unchanged, a third counted spawn refuses `ErrRecoveryExhausted` with the named way forward, and a `card_amended` re-run at two recoveries spawns uncounted; `TestBeginBatch_Record` gains the re-begin carry-over row and the remedy's two wordings, and an existing `TestRebaseline_CardSet` row over a kept batch record asserts its `Recoveries` and `RecoveryStartSHA` survive the rebaseline.
The websterengine row of `test-budget.yaml` rises by one.

**ImpactSummary:** Two new BatchState fields, one Git method with its fake, one re-signed helper with its one caller; the verbs' envelopes follow in card 25.
