# Card 3 — The prime pushes its records side only

**Edit:**
- `internal/fabricengine/pushanchored.go`
- `internal/fabricengine/weftgit.go`
- `internal/fabricengine/commit.go`
- `internal/fabriccli/weft_verbs.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/commit_integration_test.go`
- `internal/fabricengine/pushanchored_integration_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`

**Prosa:**
- `PATTERN.md`
- `docs/overview.md`

**Intent:**
D3: when the acting worktree is the hub's prime (`PrimeName(l) == l.WorktreeName`), fabric hands the code side to no push, so `lyx fabric push`, `commit`, `sync` and `lyx config`'s post-edit `sync` never publish unpushed parent-branch commits.
In every other worktree, task pairs included, both sides push as today.

- `PushPairAnchored` resolves the prime from the `Location` it holds, before taking the push lock.
  In the prime it pushes the records side only; when the code branch has unpushed commits it sets a new `PushResult` field, serialized as `code_push_skipped`, naming the branch and saying it was left for the operator to push.
- `Fabric.Commit` resolves the prime from the `Location` it already resolves from its code path, and does so before taking any lock, like its other pre-flight refusals.
  In the prime the detached push it spawns after a landed commit gets an empty code-side path, so the child pushes the records side only.
  This covers `lyx fabric commit`, `sync`, `config` and every other `Commit` caller.
- The `sync` verb in `weft_verbs.go` decides from its `Location`: in the prime it spawns the push with an empty code-side path and appends no `push_spawned` entry for the code side.
  The `push` verb adds `code_push_skipped` to its success envelope when the result carries it.
- A prime-name resolution failure fails the call before it commits or pushes anything: `PushPairAnchored` and the `sync` verb return the error, and `Fabric.Commit` returns it before taking any lock, so no commit lands and no push spawns.

What this changes, and its bound: it removes a push, never adds one; it skips only the prime's code side, on the routes above, and landing's own push of the parent branch is untouched.

Docs in this commit: the `push` and `sync` long help say the prime pushes the records side only and name `code_push_skipped`; `internal/fabricengine/doc.go`'s push and spawn text says the same; the `PATTERN-push-both-sides` line in `PATTERN.md` states that the prime's `fabric push`/`sync`/`commit` are records-only too; `docs/overview.md` changes only where it describes fabric's push or sync.

Coverage:
- `hubmutation_integration_test.go`: `PushReportsADivergedWarpSideAndStillPushesWeft` becomes a prime case with an unpushed local `main` commit: `push` exits 0, its envelope carries `code_push_skipped` naming the branch, origin's code branch is unchanged, and the records side is pushed.
- `weftskippush_integration_test.go`: the prime `sync` case expects exactly one `push_spawned` entry, for the records side.
- `commit_integration_test.go`: `TestCommit_InvokesPushRecorder` expects an empty code-side path in the prime fixture; a pair-fixture row still expects both paths; a prime-name resolution failure returns an error with no commit landed and no recorder call.
- `pushanchored_integration_test.go`: the existing pair cases keep asserting both sides pushed; a prime case asserts origin's code branch unchanged and `code_push_skipped` set; a prime-name resolution failure returns an error with nothing pushed.

**ImpactSummary:** the code-side push of `PushPairAnchored`, `Fabric.Commit`'s spawn and the `sync` verb narrows in the prime only; loom's per-transition push goes through `PushPairAnchored` and gains the same prime rule, while pairs are unchanged.
