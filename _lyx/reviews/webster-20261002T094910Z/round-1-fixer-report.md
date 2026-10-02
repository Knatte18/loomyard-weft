# Webster-Review round 1 — fixer report

All eight findings are fixed, each in its own commit.
After the last fix, the plan's whole `verify:` block passes in the worktree: `go build ./...`, `go vet ./...`, `go test ./...`, the full `-tags integration` package list, and `go vet -tags smoke ./...`.

## Fixed

- **F1 (MEDIUM): `123bf636d` hubgeom: tell the prime by its .git entry and refuse when it cannot.**
  `ReedGeometry` now tells the prime from a task worktree with `isPrimeWorktree`, which stats `<worktree>/.git` instead of spawning `git worktree list`.
  A directory, or a `gitdir:` file pointing straight at a git directory, is the main worktree (the prime).
  A `gitdir:` file pointing into `worktrees/` is a linked worktree (a task worktree).
  A missing or unreadable entry is the function's error, as card 6 requires, so a task worktree is never told a prime's geometry.
  No hub wiring path spawns git any more, so the untagged wiring tests are offline again (Test Tier Purity Invariant).
  The three Tier 1 `hubLocation` helpers (`burlercli`, `webstercli`, `loomcli`) create a `.git` directory in their fictional worktree, and their file headers say so.
  `hubgeom_test.go` gains `TestIsPrimeWorktree` (five rows) and `TestReedGeometry_PrimeAndTaskSlug`.
  `ReedGeometry`'s doc and `hubgeom/doc.go` name the stat and the error.
  This commit also replaced the worktree's uncommitted `hubgeom.go` edit, which returned the `PrimeName` error and broke 33 wiring tests.
- **F2 (LOW): `fb6d0dcd8` semantic line breaks in the new and changed comments.**
  48 comment paragraphs across 30 files were reflowed to one sentence per line, breaking a long sentence only at an independent-clause boundary.
  In legacy paragraphs, only the sentences holding changed lines were reflowed, and untouched legacy lines keep their wrap.
- **F3 (LOW): `c841675d9` driver and status strand docs state reed's held-name refusal.**
  `LoomDriverStrandName`, `statusStrandDisplayName` and `resolveStatusStrandAction` now say that a second add under a held explicit role is refused.
  They no longer say it appends a second pane.
- **F4 (LOW): `eeda57ca4` reuse aliveIDSet.**
  `paneIsLive` is removed.
  `repairNames` and `directoryRows` test liveness through `aliveIDSet`.
- **F5 (LOW): `c05efc4e6` list.go doc.**
  The doc now says each row shows the pane title and whether it has drifted from the name.
- **F6 (LOW): `eb6988638` overview.md.**
  The `reconcile` clause sits beside the warp binding it refers to again.
  The `--code` and `code` sentences follow it.
- **F7 (NIT): `5e8a40321` remove's help.**
  The paragraph is rewrapped to the surrounding width, with "exactly one of the two" placed right after the two alternatives.
- **F8 (NIT): `827a4e9f8` agentname.Resolve.**
  It uses `n.prefix()`.

## Deferred

None.
