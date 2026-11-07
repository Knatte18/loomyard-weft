# Copy hubs in fabric and adopt fixtures; hook without dirname/basename

The `fabricengine` records fixture and the `adoptcli` integration tests now start from `hubforge.CopyHub` of the anchor instead of building a fresh hub with `hubforge.NewHub`.
A fresh `NewHub` costs about 103 ms against 17 ms for a copy, and none of these tests exercise the clone or wiring.

The post-checkout hook and the chain wrapper no longer spawn `dirname` and `basename` on every checkout.
Each checkout of a test's pairs ran two extra processes.

## Changes

- `newRecordsFixture` in `internal/fabricengine/replay_integration_test.go` and the `hubforge.NewHub` sites in `internal/adoptcli` take `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})`.
- `internal/fabricengine/post-checkout.sh` derives the weft sibling with `${WORKTREE_ROOT%/*}` and `${WORKTREE_ROOT##*/}`.
- `chainedHookPreamble` in `internal/fabricengine/hook.go` derives its directory with `${0%/*}`, falling back to `.` when `$0` holds no slash.
- `TestInstallPostCheckoutHook_ChainIdempotent` now runs a real branch checkout and asserts the chained user hook's output appears, so the wrapper's new directory lookup is exercised.

## Design decisions

- `git rev-parse --show-toplevel` answers an absolute path with no trailing slash, and for such a path the expansion and the two commands agree.
- A hub that already holds the hook keeps its old script, which still works: the installer leaves a hook carrying its sentinel alone.
- The `internal/hubreconcile` sites are left alone; they are edited with another task that touches the same file.

## To check

- Each `hubforge.NewHub` site kept in `internal/fabricengine` and `internal/adoptcli` should be there because its test is about the clone or the wiring.
- The post-checkout hook still warns on a warp and weft branch mismatch and stays silent when they match.
  The existing hook tests cover this; they were not run for this description.
