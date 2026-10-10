# Card 13 — bolt-push-seed-drop-retry

**Edit:**
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine/boltpullfirst_integration_test.go`
- `internal/fabricengine/doc.go`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli#runShortname`
- `internal/fabriccli#CloneAndWire`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/fabricengine#Bolt.PushRecorded`

**Uses:**
- `internal/gitrepo#ErrPullRebaseFailed`
- `internal/fabricengine#seedCommitsAhead`
- `internal/fabricengine#boardDropRequest`
- `internal/fabricengine#dropSeedCommits`
- `internal/fabricengine#KindCommitsDropped`
- `internal/gitrepo#Repo.UpstreamSHA`
- `internal/gitrepo#PushLockFileName`
- `internal/gitrepo#Repo.CommitsNotIn`
- `internal/fabricengine#NewMutations`

**Intent:** D5's push-path rule: a seed commit that conflicts at push is dropped and the push retried once.
`Bolt.PushRecorded` delegates to `pushWeftAt` and, only when the error satisfies `ErrPullRebaseFailed`, takes the board write lock and then `gitrepo.PushLockFileName` in the board dir, which `PushCoalesced` released when `pushWeftAt` returned, lists the ahead set against `UpstreamSHA` with `CommitsNotIn`, and, when `seedCommitsAhead` finds seed commits in it, runs `dropSeedCommits` with `boardDropRequest` and the caller's `rec`, releases both locks and retries `pushWeftAt` once; the push lock keeps another process's `pull --rebase` in `_board` from interleaving with the reset and replay, and the order matches card 7's `pullFirst`, so it cannot deadlock; with no seed commits ahead, or when the drop refuses or its replay conflicts, the original error is returned as today.
What the retry can now drop is bounded by card 7's gate: only commits `IsSeedCommit` admits, only in a hub's own `_board`, never with force, every non-seed commit replayed or kept at its original tip; a second failure is returned, never retried again.
`PushRecorded` is a sibling of `Push`, not a re-sign, because `Bolt.Push(opts) error` has callers with no mutation record, board sync, `hubreconcile` and `configcli`'s hub-wide edit, whose push paths are outside this task's scope.
`Bolt.Push` keeps its signature and calls `PushRecorded` with a local `Mutations` rooted at the hub's container, logging the dropped SHAs from it at Info, so a caller without an envelope still sees the drop in the log.
Every caller that holds a mutation record for its envelope (`PATTERN-mutation-record`) switches its Bolt push to `PushRecorded` with that record, so a drop at its push shows as a `KindCommitsDropped` entry under the envelope's `mutations`: `runReconcile`'s two pushes, the hub-wide config's and the warp binding's, `runShortname`'s push after `CommitShortname`, and `CloneAndWire`'s push after the clone's anchor commit.
The comments at those pushes that say the push records nothing gain that a seed-commit drop behind it is recorded.
`doc.go` adds the push retry beside card 7's pull-first rule, under the same lock order, and states the rule: a push whose caller holds a record records the drop there, and `Bolt.Push` only logs it.
Tests in `boltpullfirst_integration_test.go`: a new `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` with a seed and a board commit ahead of a moved upstream landing the board commit after the drop with the passed `rec` holding one `KindCommitsDropped` entry naming the seed commit, and a conflicting board commit leaving Bolt at its tip with the error returned and nothing recorded; the budget row for `internal/fabricengine` rises by one.

**ImpactSummary:** `Bolt.Push` gains a retry path reached only on a rebase conflict with seed commits ahead, so every caller keeps its contract otherwise; the pushes of `runReconcile`, `runShortname` and `CloneAndWire` move to the recording sibling, adding a `commits_dropped` entry to the `mutations` of `lyx fabric reconcile`, `lyx fabric shortname` and `lyx fabric clone` only when a drop happened.
