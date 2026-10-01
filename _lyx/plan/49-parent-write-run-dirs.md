# Card 49 — parent-write-run-dirs

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/gitwrap_test.go`

**Intent:**
Fix PR #321 round-3 finding D: two kinds of Master write class as policy although each can change what a run decides.

- A write to webster's own pause flag or a `*.lock` lives under `Geom.ScratchDir`, which `ClassifyViolation`'s `runDirs` list leaves out, so it falls through to the git-ignored rule and warns.
- A write into another lyx worktree of the hub (a sibling task worktree, or the prime) lies outside this run's worktree, so the `!pathWithin(worktree, written)` branch classes it as policy.

In `audit.go`'s `ClassifyViolation`:

- `runDirs` gains `geom.ScratchDir`.
- After the `runDirs` loop and before the outside-the-worktree branch, a write that lies within another worktree of the task repository, or within that worktree's `_lyx` (link-resolved, joined with `lyxdirs.LyxDirName`), is correctness.
  "Another" means any worktree whose canonical root is not this run's canonical `WorktreeRoot`.
- The doc comment names both rules.

In `gitwrap.go`, a new unexported `otherWorktrees(worktree string) ([]string, error)` runs `git -C <worktree> worktree list --porcelain` and returns the canonical root of every listed worktree except `worktree` itself.
Its error is the git probe's, which `ClassifyViolation` already returns as its only error.
It reads the warp repository's own worktree list, so websterengine stays fabric-blind and derives no hub path.

`doc.go`'s audit-findings paragraph adds both writes to the correctness examples.

Tests:

- `gitwrap_test.go`: `TestOtherWorktrees` over `gitwrapNewScratchRepo` with one added worktree (`git worktree add`) returns exactly that worktree's canonical root from the main checkout, and exactly the main checkout's from the added one.
- `auditseverity_test.go`: `TestClassifyViolation` sets `ScratchDir` on both geometries and gains rows:
  a write to `<ScratchDir>/pause` and one to `<ScratchDir>/mutate.lock` are correctness;
  a write to a tracked file of a second worktree added with `git worktree add`, and one under a `_lyx` link inside that worktree pointing to a temp directory, are correctness;
  the existing `outside worktree` row, a temp directory that is no worktree, stays policy.

**ImpactSummary:** Master writes under webster's scratch directory or into another worktree of the task repository become correctness findings; writes elsewhere outside the worktree stay policy.
