# Card 11 — preflight-detached-push

**Create:**
- `plan:internal/boardengine#SpawnSync` -> `func SpawnSync(boardPath string) error`
- `plan:internal/hubreconcile#spawnBoardSync` -> `var spawnBoardSync func(boardPath string) error`
- `plan:internal/hubreconcile#SetSpawnBoardSyncForTest` -> `func SetSpawnBoardSyncForTest(spawn func(boardPath string) error) (restore func())`

**Edit:**
- `internal/hubreconcile#reconcileHubWide`
- `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`
- `internal/hubreconcile/doc.go`

**Intent:** D11: the Preflight row's synchronous records push goes detached.
The Preflight row's only synchronous push of the records repository is `hubreconcile.Ensure`'s `reconcileHubWide`, which ends in `bolt.Push`, a foreground push to GitHub of the board commit the hub-wide config write made, so a first push that origin refuses with `(failed)` holds the row for the whole backoff.
`reconcileHubWide` now commits through `PullThenCommitWritten` as today and then calls `spawnBoardSync` (defaulting to `boardengine.SpawnSync`), the detached `lyx board sync` every board write already spawns, and returns at once.
`boardengine.SpawnSync` is the exported entry over the existing `spawnSync`, returning nil without spawning in a test binary, where forking the running binary would re-run the suite.
The build stamp no longer waits on the push: a hub-wide commit that landed locally writes the stamp, a pull-first skip still leaves it absent, and a refused detached push reaches the orch as card 9's failed-push record instead of keeping the stamp absent.
`hubreconcile/doc.go`'s push paragraph is rewritten to say so.

`SetSpawnBoardSyncForTest` lives in `hubreconcile`'s `export_test.go`, beside `EnsureWithHooks`, since the test is in package `hubreconcile_test`; the swap is package-global, so the test that makes it runs without `t.Parallel`.
`TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` replaces `spawnBoardSync` with a recorder through it and asserts it was called once with the board dir after the hub-wide commit, and that no synchronous push ran (the records bare's board branch is unchanged when the walk returns).
Grep `internal/hubreconcile`'s tests for assertions that a failed board push leaves the stamp absent and update each to the detached behaviour.

**ImpactSummary:** One unexported function in hubreconcile changes its push to a spawn; Preflight and every start verb that calls `Ensure` return without a network push.

**Commit:** 11: hubreconcile: the Preflight row's board push goes detached
