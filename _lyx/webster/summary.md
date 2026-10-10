# test-suite-weight: all batches done

This session resumed with batches 01 and 06 done and batch 08 failed, then drove the remaining batches to `done`.

- **08-reedcli-probes-onto-tmuxkit:** `recover-batch` finished `done` at `d44ca24f4`, with `internal/testkit/tmuxkit/allow_none.go` and `allow_tmux.go` as deviations.
- **14-claudeengine-prebuilt-deny:** done at `5665db59d`.
  The spawned-agent deny covers `LYX_PREBUILT_LYX`, and the hermetic git config disables detached gc.
  `hubforge` gained shapes, `CopyHub` and `SharedHub`, and the first third of `fabricengine`'s hub-taking tests moved onto the template model.
  Deviations: `settings.go`, `hookcmd_integration_test.go`, `hermetic.go`, `hub.go`, `template.go` and three `pattern/` files.
- **18-reclassify-fabricengine-head-to-push:** done at `5fdd2b1f8`.
  The middle and last thirds of `fabricengine`'s tests moved onto `CopyHub`.
  Three tests stayed on `NewHub`, because a relocated copy leaves the `_board` worktree dirty.
  Deviation: `livestate_matrix_test.go`, which the fork's report names although it is a card target.
- **20-reclassify-fabriccli-and-geometry:** done at `52a4b518a`.
  The `fabriccli` and geometry tests, and the loom, landing, webster, shed and batten tests, moved onto `CopyHub`.
  Three tests stayed on `NewHub` for the same dirty-`_board` reason.
  No deviations.
- **22-reclassify-reed-orch-shuttle:** done at `279fbfe09`.
  The reed, orch, shuttle, seat, treadle and burler tests moved onto `CopyHub`.
  The new `internal/dotgit` leaf parses `.git`, and `lyxcwd` finds the worktree root through it and spawns no git.
  Deviations: `configcli_integration_test.go`, `stencilseed_integration_test.go`, `gitexec/doc.go` and `lyxcwd/anchor.go`.
- **25-gitrepo-geometry:** done at `3d8a43bbe`.
  `gitrepo` reads geometry through `dotgit` and ref reads through go-git, with parity rows.
  Git children lose `GIT_DIR` and `GIT_WORK_TREE`, and `fabricengine`'s spawns moved onto `gitrepo`.
  Deviations: the new `internal/gitrepo/refreads.go` and `cmd/lyx/checkedcall_test.go`.
- **31-tierpurity-tmux-constraint:** done at `d37398c62`.
  The tier-purity scan bars tmuxkit spawns from files that are not `tmux` or `llm` tier, and a scan pins `PATTERN-shuttle-stop`.
  The after-state report and comparison are in `docs/benchmarks/test-suite-weight.md`.
  Deviation: `internal/lyxcwd/enforcement_test.go`, which gained an allowlist entry for the parity oracle's `--show-toplevel`.

The fork's report for batch 31 says two packages fail in the after-state measurement and that it did not fix them.
`internal/gitrepo` tier 2 fails `TestPushCoalesced` because the nested tmuxkit directory pushes the tmux socket path past 107 bytes.
`internal/reedcli` tier 3 reported a leftover `sleep 300` child.
Both are marked `FAIL` in the report.
