# loom start: commit the friction clear so Preflight stays clean

A task worktree branched from a fabric that carries committed friction notes, such as a beam pawl run's own `loom-halt.md`, no longer blocks at Preflight.
`lym loom start` and `step` now commit the friction directory along with the seed.

Before, a task's first seed deleted the inherited notes but never committed the deletion.
Preflight's worktree-clean check then refused on the uncommitted `D .../_lym/loom/friction/loom-halt.md`, and the run halted blocked.

Closes #22

## Design decisions

- The fix is the issue's second option: the child's start clears the inherited notes and commits that removal itself.
  The beam's weft still commits its own friction notes, so the first option, not committing them, is left alone.
- `seedAndCommitBootstrap` adds the friction directory to its commit pathspec on every invocation, not only the first seed.
  This self-heals a clear whose commit crashed, and it commits a resume's notes, which stay on disk.
- Git refuses a pathspec that matches nothing, so the directory is added only when the new `fabricengine.AnchoredPathHasContent` finds tracked or untracked, non-ignored content.
  A tracked file deleted on disk counts, since that deletion is what the commit must carry.
- The probe is skipped under `SkipGit` and when no friction directory is configured, where the commit is a no-op and the fabric's git may be unusable.
- `bootstrapCommitPaths` takes a `withFriction` flag so the pathspec shape stays a pure, tier-0 testable function.
- `docs/overview.md` and the `lym loom start` description in `internal/loomcli/doc.go` state the new behavior.

## Tests

- `TestAnchoredPathHasContent` covers a tracked file deleted on disk, an untracked file, an ignored-only directory and an empty directory.
- `TestBootstrapCommitPaths` covers the pathspec with and without the friction directory.
- `TestSmokeStepBootstrapWiring` now commits the stale note before the first step and asserts the run does not end blocked or failed.
  It also asserts a resume's note is tracked after a re-entry.

## Check by hand

- None of the tests above were run as part of writing this description; run them through `lym gate test ./internal/fabricengine ./internal/loomcli`.
- Optionally reproduce the issue: commit a `loom-halt.md` into a beam's weft, release a task in the beam, and confirm its first `lym loom start` passes Preflight.
