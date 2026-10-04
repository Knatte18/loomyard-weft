# Fixer report — plan, round 1

All six findings are fixed; nothing is deferred.

## Changes

- **F1** (`02-pattern-warp-root.md`, `00-overview.md`):
  - Card 2 now gives `burlerengine.Geometry` and `websterengine.Geometry` a told `RepoRoot` field.
    The hub and standalone geometry constructors fill it, and their files and tests are added to the Edit list.
  - Burler passes `e.geom.RepoRoot`.
    `RenderMasterPrompt` and `RenderRecoveryPrompt` replace `anchorRoot` with `repoRoot`, and the recovery render keeps `promptWorktreeRoot` for the card pointers.
  - Subpath-anchored tests are added for burler and webster, and the `ImpactSummary` is updated.
  - The `directive-root` Shared Decision now says call sites pass the repo root and why burler and webster need a `RepoRoot` field.
- **F2** (`02-pattern-warp-root.md`): the Intent states the bound on the whitespace-only rule, with card 6's real-file test as the guard.
- **F3**:
  - Card 2 corrects the stale `_lyx/PATTERN.md` comments in `unwire.go` and `junction.go`.
  - Card 1 adds `tools/sandbox/SANDBOX-FABRIC-SUITE.md`, corrects `fabricengine/doc.go`'s second PATTERN-path sentence and adds a `_lyx/PATTERN` grep.
  - Card 13 rewrites `docs/overview.md`'s `internal/pattern` row and sentence for the new rule.
- **F4**: removed the 14 files whose only hits are `vocab-exempt` identifiers from cards 14, 15 and 16.
- **F5**:
  - Card 18 states that `internal/configsync` keeps its narrow carve-out for its test files, that every other owner directory is skipped whole, and that the `sandbox/` skip is the repo-root tree only.
  - Card 14 adds `internal/configsync/configsync_test.go` with an identifier-only rename.
- **F6** (`20-spawn-role-constants.md`): the scan names `reedengine.AddSpec`.

## Deferred

None.
