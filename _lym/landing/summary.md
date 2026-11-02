# Reed buttons run in the worktree's current anchor

A Reed status-bar button, the "code" button for one, failed with a wrong-anchor-path error when its worktree's anchor was not `.` and the anchor marker had changed after the session last rendered.
The click ran the button's command in `spec.Dir`, the directory frozen into the session's `@lym_button_spec` option at boot or attach.
`lym ide spawn` and the other lym verbs gate their cwd on `<worktree>/<anchor>`, so a directory that still existed but was no longer the anchor was refused.
The click checked only that `spec.Dir` existed.

Now a click reads the anchor afresh and runs the entry there.
No reattach is needed after an anchor marker changes.

Closes #10

## What changes

- `lym reed button` resolves the worktree holding the rendered `spec.Dir` through `lymcwd.ResolveWorktree` and takes its `AnchorPath()` as the run directory (`buttonRunDirectory` in `internal/reedcli/button.go`).
  The resolve applies no cwd gate and spawns no git.
- `reedengine.RunButton` now takes that reader as a second argument.
  The entry runs in the anchor directory, and the confirm prompt's confirmed line opens in it too.
- A click shows one message on the clicking client and runs nothing when the anchor does not resolve, or when the anchor directory is gone.
  The gone-anchor message tells the operator to point the worktree's anchor at an existing directory.
- When the current anchor differs from the rendered directory, the click logs that at Info with both paths.
- Docs and comments in `reedcli` and `reedengine` describe the new run directory.

## Design decisions

- This is the fix for candidate cause 1 of the issue, a stale spec.
  The cause was confirmed by a unit test that records an anchor marker after the render and then asks where the click runs; the parent-anchor candidate was not touched.
- The anchor is read at click time instead of treating a failing `spec.Dir` as stale and asking for `lym reed attach`.
  The operator's click works at once, and the rendered `spec.Dir` is still the key that finds the worktree.
- The button verb still reads no cwd, config or reed state, so a `reed.yaml` that fails to load cannot abort a click.

## Tests

- `TestButtonRunDirectory` in `internal/reedcli/button_test.go` covers a root render with a subpath marker recorded later, a subpath render with a root marker recorded later, a render that matches the marker, no marker, and a directory in no worktree.
- `TestRunButtonVia` gains rows for a moved anchor (plain run and confirm prompt), a gone anchor directory and an unresolvable anchor.
- Not run by hand: a click in a real tmux session whose anchor was changed after attach.
  Check that the "code" button opens the IDE after `lym fabric anchor --set`, with no reattach.
