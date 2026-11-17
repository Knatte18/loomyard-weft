# Card 3 — reconcile-renders-file-targets

**Create:**
- `plan:internal/fabricengine#NotifyWorktreeMadeAt` -> `func NotifyWorktreeMadeAt(worktreePath string) string`
- `plan:internal/hubreconcile_test#TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree` -> `func TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree(t *testing.T)`

**Edit:**
- `internal/hubreconcile#reconcileWorktree`
- `internal/fabricengine_test#TestWorktreeSeam_Events`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/hubreconcile/doc.go`
- `pattern/PATTERN-heddle-instructions.md`

**Uses:**
- `internal/fabricengine#NotifyWorktreeMade`
- `internal/fabricengine_test#fakeWorktreeSeam`
- `internal/hubreconcile_test#newStaleHub`
- `internal/hubreconcile_test#geometryOf`

**Intent:**
The hub config reconcile, which runs once per lym build, renders heddle's file targets into every live worktree of the hub (D7),
so a shipped-entry change in a new build reaches existing worktrees without a worktree event or a `lym heddle` write.

`hubreconcile` may not import `internal/lymcwd` (`TestToldGeometryInvariant_AllowlistOnly`), so fabric gains `NotifyWorktreeMadeAt(worktreePath)`.
It resolves the worktree at `worktreePath` through `lymcwd.ResolveWorktree` and fires the installed seam's `Made` as `NotifyWorktreeMade` does, returning its warning text.
With no seam installed it returns "" and resolves nothing.
A path that does not resolve returns a warning text through `seamWarning`, which logs at Warn and names `lym heddle render` in that worktree as the way forward; it is never an error.

`reconcileWorktree` calls `fabricengine.NotifyWorktreeMadeAt(w.Path)` for each worktree it does not pass over as incomplete or removed,
right after `OpenCodeWorktree` succeeds and before the merge check, since the rendered files are git-excluded and no commit or merge touches them.
The walk reaches the prime and every pair, and the single-pair call reaches its pair.
A failed render is the seam's logged warning and never fails or skips the walk.
In the lym binary the seam is heddle's, installed in `cmd/lym/main.go`.

Tests:
- `TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree` is a tier1 integration table in `hubreconcile_test`.
  It is a test of its own, not an assertion in `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, because it installs the process-wide seam with `fabricengine.InstallWorktreeSeam` and restores it in cleanup, so it is not parallel.
  Each row builds the hub with `newStaleHub` and removes `CLAUDE.local.md` and `AGENTS.override.md` from each code worktree's anchor `fabricengine.CodeWorktrees` lists.
  - `StaleWalk` installs `heddleengine.Seam{}`, runs `hubreconcile.Ensure(geometryOf(h), …)`, and asserts each anchor again holds both files, each opening with `heddleengine.Marker`.
  - `PairCall` installs `heddleengine.Seam{}`, runs `Ensure` with `Options.Pair` naming one pair, and asserts that pair's anchor holds both files and the prime's holds neither.
  - `FailingSeam` installs a seam whose `Made` returns an error, and asserts `Ensure` returns nil and writes the build stamp.
- `TestWorktreeSeam_Events` gains two subtests over its `fakeWorktreeSeam`:
  `NotifyWorktreeMadeAt` on a hub worktree fires one made event for it and returns "";
  on a path in no worktree it fires nothing and returns a warning naming `lym heddle render`.
  The no-seam case is its subtest that runs before the seam is installed, which gains a `NotifyWorktreeMadeAt` call returning "".

`PATTERN-heddle-instructions.md`'s heddle-rendering list names the hub config reconcile, once per lym build, as a file-target render site beside fabric's seam, `lym heddle render` and `lym heddle set` and `remove`.
`hubreconcile/doc.go` says the walk renders each walked worktree's file targets through fabric's seam;
`fabricengine/doc.go` names `NotifyWorktreeMadeAt` beside `NotifyWorktreeMade`.

**ImpactSummary:** one added call in the per-worktree reconcile step and one new fabric function wrapping the existing seam fire; no signature changes, and a render failure stays a logged warning.
