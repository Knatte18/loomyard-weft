# Card 3 — The prime pushes its records side only

**Edit:**
- `internal/fabricengine/pushanchored.go`
- `internal/fabricengine/weftgit.go`
- `internal/fabricengine/commit.go`
- `internal/fabricengine/coalesce.go`
- `internal/fabriccli/weft_verbs.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/commit_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`
- `internal/webstercli/sync_integration_test.go`

**Prosa:**
- `PATTERN.md`
- `docs/overview.md`

**Intent:**
D3: when the acting worktree is the hub's prime (`PrimeName(l) == l.WorktreeName`), fabric hands the code side to no push, so `lyx fabric push`, `commit`, `sync` and `lyx config`'s post-edit `sync` never publish unpushed parent-branch commits.
In every other worktree, task pairs included, both sides push as today.

- `PushPairAnchored` resolves the prime from the `Location` it holds, before taking the push lock.
  In the prime it pushes the records side only; when the code branch has unpushed commits it sets a new `PushResult` field, serialized as `code_push_skipped`, naming the branch and saying it was left for the operator to push.
- `Fabric.Commit` resolves the prime from the `Location` it already resolves from its code path, directly after that resolution and before taking any lock, like its other pre-flight refusals.
  In the prime the detached push it spawns after a landed commit gets an empty code-side path, so the child pushes the records side only.
  This covers `lyx fabric commit`, `sync`, `config` and every other `Commit` caller.
- The `sync` verb in `weft_verbs.go` decides from its `Location`: in the prime it spawns the push with an empty code-side path and appends no `push_spawned` entry for the code side.
  The `push` verb adds `code_push_skipped` to its success envelope when the result carries it.
  `lyx config`'s post-edit `sync` is this same verb, run in-process through `fabriccli.RunCLIIn`, so it takes the rule with no change of its own in `configcli`.
- A prime-name resolution failure fails the call before it commits or pushes anything: `PushPairAnchored` and the `sync` verb return the error, and `Fabric.Commit` returns it before taking any lock, so no commit lands and no push spawns.

What this changes, and its bound: it removes a push, never adds one; it skips only the prime's code side, on the routes above, and landing's own push of the parent branch is untouched.

Docs in this commit: the `push` and `sync` long help say the prime pushes the records side only and name `code_push_skipped`; `internal/fabricengine/doc.go`'s push and spawn text says the same; the `PATTERN-push-both-sides` line in `PATTERN.md` states that the prime's `fabric push`/`sync`/`commit` are records-only too; `docs/overview.md` changes only where it describes fabric's push or sync.
The comments and strings that state a both-sides push change with it:
- `weft_verbs.go`'s file header and the `push` and `sync` `Short` strings;
- `commit.go`'s `Commit` doc, which says it fires an async both-sides push;
- the `coalesce.go` sentence saying the detached push child always supplies both paths, since in the prime it supplies the records path alone.

Coverage:
- `hubmutation_integration_test.go`, two existing steps run `push` from the prime and both pin the old behaviour:
  `PushReportsADivergedWarpSideAndStillPushesWeft` becomes the one prime case, with an unpushed local `main` commit: `push` exits 0, its envelope carries `code_push_skipped` naming the branch, origin's code branch is unchanged, and the records side is pushed.
  `PushPushesBothSides` becomes the task-pair case, since it would otherwise assert the prime's code side is pushed: the step adds a pair, commits on both of the pair's sides, runs `push` from the pair's code worktree, asserts both bares reach their local HEADs and the envelope carries no `code_push_skipped`, and leaves the hub as the later steps expect it.
- `weftskippush_integration_test.go`: the prime `sync` case expects exactly one `push_spawned` entry, for the records side, and a task-pair `sync` row expects two, one per side; this pins `lyx config`'s route too.
- `commit_integration_test.go`: `TestCommit_InvokesPushRecorder` gains `t.Run` rows: the existing plain fixture is the prime, so its row expects an empty code-side path; a hubforge pair row expects both paths; a prime-name resolution failure row returns an error carrying `resolve main worktree`, with no commit landed and no recorder call.
- `pushanchored_integration_test.go`: the existing pair cases keep asserting both sides pushed; `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` gains `t.Run` rows for a prime Location, asserting origin's code branch unchanged and `code_push_skipped` set, and for a prime-name resolution failure, returning an error with nothing pushed.
- The fabricengine cases are rows of those two existing top-level tests and add no top-level test function, so `internal/fabricengine`'s budget row in `cmd/lyx/testdata/test-budget.yaml`, already at its count, stays untouched; the `fabriccli` cases are steps and `t.Run` rows of existing scenarios.
- Two existing tests build a `Commit` call whose anchor names a subdirectory absent from the code worktree, which the new prime resolution refuses because `PrimeName` runs `git worktree list` in `l.AnchorPath()`.
  Their fixtures are corrected, and no case is added:
  `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` in `commit_integration_test.go` creates `<warpPath>/wts/some-task` before it calls `Commit`;
  `newPairFixtureAt` in `internal/webstercli/sync_integration_test.go` creates `<code>/<relPath>` in the code worktree, which the `one segment`, `two segments` and `three segments` rows of `TestFabricSync_CommitsAtEveryRelPathDepth` need.
- A prime-name resolution failure is made reachable without a seam: `PrimeName` fails when `git worktree list` fails in `l.AnchorPath()`, so the `PushPairAnchored` row passes a `Location` whose `AnchorRel` names a subdirectory absent from the worktree, and the `Commit` row records an anchor naming such a subdirectory in the fixture hub's board directory, the marker `lyxcwd` reads, before it calls `Commit`.

**ImpactSummary:** the code-side push of `PushPairAnchored`, `Fabric.Commit`'s spawn and the `sync` verb narrows in the prime only; loom's per-transition push goes through `PushPairAnchored` and gains the same prime rule, while pairs are unchanged; `Commit` now also fails for an anchor subdirectory missing from the code worktree, which a hub built through `hubforge` never has and two hand-built test fixtures did, so those two fixtures change.
