# Card 8 — Neutral file names and Fabric handle fields

**Rename:**
- `internal/fabricengine/warpprobe.go` -> `internal/fabricengine/recordsprobe.go`
- `internal/fabricengine/warpbinding.go` -> `internal/fabricengine/codebinding.go`
- `internal/fabricengine/weftgit.go` -> `internal/fabricengine/recordsgit.go`

**Edit:**
- `internal/fabricengine/fabric.go`
- `internal/fabricengine/commit.go`
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/diff.go`
- `internal/fabricengine/index.go`
- `internal/fabricengine/merge.go`
- `internal/fabricengine/mergeguards.go`
- `internal/fabricengine/mergelifecycle.go`
- `internal/fabricengine/mergestage.go`
- `internal/fabricengine/mergestate.go`
- `internal/fabricengine/pull.go`
- `internal/fabricengine/remoteonly.go`
- `internal/fabricengine/revert.go`
- `internal/fabricengine/warpforward.go`
- `internal/fabricengine/spawn.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/export_test.go`
- `internal/fabricengine/commit_gating_integration_test.go`
- `internal/fabricengine/commit_partial_integration_test.go`
- `internal/fabricengine/pull_integration_test.go`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/snapshot_integration_test.go`
- `internal/fabricengine/weftgit_pathspec_integration_test.go`
- `internal/fabricengine/pathspec_tolerance_coupling_integration_test.go`
- `internal/fabriccli/cli_test.go`
- `//cmd/lyx/destructiveguard_test.go`
- `//cmd/lyx/uncontainedwrite_test.go`

**Intent:**
D5, file and handle part: the spellings the `cmd/lyx` guards carry outside fabric's owner set become side-neutral, while each guard keeps failing on what it bans.

- `warpprobe.go`, `warpbinding.go` and `weftgit.go` move by `git mv` to `recordsprobe.go`, `codebinding.go` and `recordsgit.go`, each file's header comment opening with its new name.
  Every reference to an old file name follows: the allowlist keys in `cmd/lyx/destructiveguard_test.go` and `cmd/lyx/uncontainedwrite_test.go`, and the comments in `fabricengine/doc.go`, `spawn.go`, `export_test.go`, `pathspec_tolerance_coupling_integration_test.go` and `fabriccli/cli_test.go`.
- The `Fabric` struct's raw `gitrepo.Repo` handle fields `warp` and `weft` become `code` and `records`, renamed with an AST-aware rename across the package and its tests; the Edit list is every file that reads either field.
  The path fields stay.
- `cmd/lyx/destructiveguard_test.go`'s banned tokens `warp.ResetHard(` and `weft.ResetHard(` become `code.ResetHard(` and `records.ResetHard(`, and its comment names the code-side and records-side handles.
  The guard still fails on a raw `ResetHard` through a `Fabric` handle field in any fabricengine file but `destroy.go`, and still passes `Fabric.ResetHard`, the gated method.
- `uncontainedwrite_test.go`'s allowlist reasons stay worded as they are, with only their keys changed.

A rename with no behaviour change: the two guards, which walk the renamed files, and the package's tests are its coverage.

**ImpactSummary:** fabricengine-internal only: three file names and two unexported struct fields change, plus the two `cmd/lyx` guards' file keys and banned tokens that track them.
