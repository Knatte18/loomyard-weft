# fabricengine: name the anchored directory in lym way-forward lines

In a hub whose anchor is a subdirectory, the way forward of a failed-push notice named the worktree root.
Every `lym` verb refuses a cwd other than the anchored directory, so the command failed as written with "cwd is outside the recorded fabric anchor subtree".
A `lym` step in a way forward now names the code worktree's anchored directory, and the line works as written.

Closes #98

## Changes

- `lymVerbDir` in `internal/fabricengine/failedpush.go` returns the anchored directory of a code worktree.
  It reads the worktree's recorded marker when the line is built, so a record written before an anchor change names the current anchor.
  A worktree with no marker keeps its root.
  A marker that cannot be read also keeps the root, logged at Warn.
  An empty worktree stays empty.
- `pushStep`, `codeDivergedWayForward` and `rewoundWayForward` route their `lym fabric push` steps through it.
- The reconcile stops in `originreconcile.go` do the same for `lym fabric pull`, `lym fabric anchor` and `lym fabric commit`.
- `weftDirtyStop` and the records-conflict stop in `pull.go`, and `upstreamAnchorStop` in `anchorstop.go`, do the same.
- `BranchRewritten` now names the pull in the code worktree's anchored directory, taken from `b.CodeWorktree`, where it named `b.Worktree`.
- `doc.go` states the rule.

## Design decisions

- `git` steps keep the worktree root.
  Git has no anchor gate, and the root is where the checkout lives.
  Within one line, such as the rewind way forward, the `git` steps name the root and the `lym` step names the anchored directory.
- The anchor is resolved when the line is built, not when the failure is recorded.
  Persisted failed-push records are unchanged.

## Also in this diff

`internal/gatecli/redundancy_integration_test.go`: the broken-build redundancy subtest's second unit goes from 50ms to 500ms.
A loaded machine could time a real run out at the old value.

## To check by hand

- `lymverbdir-calls-lack-anchored-tests`: confirm the `lymVerbDir` call sites outside `pushStep`, in `originreconcile.go`, `pull.go` and `anchorstop.go`, have anchored-hub test coverage.
  The tests added in `failedpush_test.go` cover the failed-push and rewind lines.
- `unconverted-lym-steps-weftdirtystop-upstreamanchorstop`: confirm no `lym` step in a way forward still names a worktree root.
  `weftDirtyStop` and `upstreamAnchorStop` were converted after this finding was raised.
- Run a failed async push in an anchored hub, such as one whose anchor is `src/csharp/NORCE.Models`, and paste the notice's `lym fabric push` line.
