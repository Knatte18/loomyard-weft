# Teardown deletes the leftover worktree directory; cancel clears it

A finished run left its task worktree directory behind, holding only ignored build output, because a build or language server refilled the path after `git worktree remove`.
That directory was no worktree, yet it blocked the slug's next pair, and `lym batten cancel --remove` died with `not a git repository` instead of clearing it.
Only a manual `rm -rf` unblocked the board write.

Closes #54
Closes #52

## What changes

- Worktree-Teardown and `lym batten cancel --remove` now delete a directory that is no worktree at the pair's task worktree path, ignored build output included.
  Both set the new `pairteardown.Request.ClearStray`, which runs after the removal left the pair gone.
- A delete that fails now fails loudly.
  `fabricengine.Remove` finishes the teardown and returns an error wrapping the new `ErrStrayPathKept`, naming the directory and the way forward, instead of logging a warning and succeeding.
  A kept directory no longer respawns the pair's push, since it leaves nothing to push.
- `batten` rewords that error as a resume instruction (`lym batten run <slug>`).
- `PairParts` gains `StrayPath`, so `lym batten cancel` counts a stray directory as something left.
  Without `--remove` it refuses and names `--remove`; with `--remove` it proceeds, and it no longer reports "nothing left" while the directory stands.
- The cancel output gains `stray_directory_deleted`, and `lym fabric remove` output gains `stray_path_deleted` when a directory was deleted.
  This replaces the old cancel warning that told the operator to delete the directory by hand.
- Beam landing reads a member's `HasPair` through `fabricengine.PairWorktreeStands`, so a leftover directory reads as no pair.

## Design decisions

- `Topology.RemoveStrayTaskPath` is a second exception to the destruction chokepoint, beside `removeWorktreeLeftover`, in `destroy.go`.
  No git command runs before it, so ownership comes from the caller: a run that created the pair at that path, or an operator who asked to remove the slug's pair.
  It refuses a registered linked worktree and any directory holding a `.git` entry, and keeps containment through `os.Root`.
  `--force` and dirtiness do not apply, since nothing there is a worktree.
  `PATTERN-fabric-git.md` and the `fabricengine` package doc state the exception.
- The delete is opt-in per caller through `ClearStray`, so no other teardown route deletes an unowned directory.
- `lym fabric remove` itself still never deletes a directory that stood at the path at entry; it deletes only one refilled after its own removal, and reports one it cannot.

## To check by hand

- Run `lym batten cancel <slug> --remove` against a slug whose task worktree path holds a plain directory, and confirm it deletes the directory and clears the board's has-a-pair block.
- Run a full batten run to Worktree-Teardown with a build writing into the worktree, and confirm no directory stays.
- The stray delete bypasses the shared four-check destruction pipeline (`stray-delete-bypasses-destruction-gate`, in `fabricengine/destroy.go`, `RemoveStrayTaskPath`).
  Confirm the caller-proved ownership and the `.git` and registered-worktree refusals are enough.
- Beam landing's member listing has no test that a leftover directory reads as no pair (`beamland-member-listing-untested`, in `internal/battencli/beamland.go`).
- `PairParts.StrayPath` is not tested through `inspectPair` (`pairparts-straypath-untested-through-inspectpair`, in `internal/fabricengine/pairparts.go`).
- `ClearStray` in the teardown request has no test that batten's Worktree-Teardown sets it (`teardown-request-clearstray-untested`, in `internal/battencli/wire.go`).
