# Card 2 — scratch-exclude

**Create:**
- `internal/fabricengine#scratchDirName`
- `internal/fabricengine#seedScratchExclude`

**Edit:**
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine_test#TestDotLyxJunction_LifecycleWiresSeedsBothExcludesAndUnwires`

**Uses:**
- `internal/fabricengine#ExcludeAnchoredDir`

**Intent:**
D1's exclusion: every warp repo fabric wires ignores the scratch place the edit directive names, so an agent's scratch files never show as untracked dirt.

`scratchDirName` and `seedScratchExclude` live in `internal/fabricengine/gitexclude.go`, beside `ExcludeAnchoredDir`.
`scratchDirName` is the directory name the edit directive names; its doc comment says `.scratch/` is agent scratch, not lyx state, which is why it is declared here and not in `internal/lyxdirs`.
`seedScratchExclude` calls `ExcludeAnchoredDir(worktreeDir, ".", scratchDirName)`, which writes the root-anchored `/.scratch/` line under the exclude lock, or nothing when `git check-ignore` already reports the directory ignored.
When it wrote, it records `KindFileWritten` at the returned exclude path on rec, as `seedGitExclude` does, so the wiring verb's mutation record names the write; a nil rec records nothing.

`WireJunctionsWith` calls `seedScratchExclude(rec, WorktreePath(l, slug))` after `seedGitExclude`, returning its error.
Unwiring leaves the line in place: the exclude file is repo-wide, so a sibling worktree still needs it, and an entry for an absent directory hides nothing.

Bound: untracked files under the warp root's `.scratch/` no longer show in `git status`, so a dirty-tree check no longer sees them.
A `.scratch/` below the root is not excluded and still shows as untracked, and the entry never hides tracked content, since git ignores no tracked path.
The comment on `seedScratchExclude` states this bound.

`TestDotLyxJunction_LifecycleWiresSeedsBothExcludesAndUnwires` asserts the warp's exclude lines hold `/.scratch/` after `WireJunctions`, and still hold it after `UnwireJunctions`.
Its doc comment and its `testtiming:keep` reason name the scratch line too.

**ImpactSummary:** One more exclude write in the wiring path every fabric wire runs; it touches only the shared exclude file under its existing lock and appends one mutation entry when it writes.
