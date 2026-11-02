# Run a reed status-bar button in its worktree's current anchor

Fixes [issue #10](https://github.com/Knatte18/loomyard/issues/10): the "code" button reported a wrong anchor path although the operator stood in the anchor directory.

## Cause

The click runs the entry, `lym ide spawn {slug}`, as a child process whose cwd is the `dir` frozen into `@lym_button_spec` at the session's last boot or attach.
`lym ide spawn` resolves its cwd through `lymcwd.Resolve`, whose gate requires the cwd to equal `<worktree>/<anchor>` exactly.
Nothing re-renders the spec when an anchor marker changes, and the click checked only that the stored directory still existed.
So a marker recorded or changed after the render left the click running in the old directory, which exists but is no longer the anchor, and the child refused it.
`lym fabric anchor --set` refuses while the worktree's own reed holds a session, but it does not cover a task session, whose button runs in its beam's or the prime's anchor, nor a marker a reconcile writes.
The operator's own terminal plays no part: the message comes from the button's child.

The second candidate from the issue, a parent whose recorded anchor differs from a fresh resolve, has no producing code path, so it is not addressed separately; the fix covers it anyway, since the click now always resolves afresh.

## Change

- `reedengine.RunButton` takes an anchor reader, and the click runs the entry, and composes the confirmed prompt's line, in the anchor that reader returns for the rendered directory instead of the rendered directory itself.
  A reader error shows `lym: <label>: the anchor of <dir> does not resolve: <err>` and runs nothing.
  A rendered directory that differs from the fresh anchor is logged at Info with both paths.
- `reedcli`'s `button` verb supplies the reader, `buttonRunDirectory`, through `lymcwd.ResolveWorktree(dir).AnchorPath()`: no cwd gate, no git spawn, the worktree found from any directory inside it.
  reedengine still never imports lymcwd.
- The render, the `@lym_button_dir` cd in the click line, and the gone-directory check are unchanged.

## Tests

- `TestRunButtonVia` gains rows for a moved anchor (the plain run and the confirm prompt's line both use the fresh anchor) and an unresolvable anchor.
- `TestButtonRunDirectory` (new, reedcli, tier 0) records an anchor marker after a render at the root and at a subpath and checks where the click runs, plus a directory in no worktree.
  It records the hub-wide Bolt marker, since the fabric-vocabulary guard keeps `weftname` out of reedcli; both markers go through the same reader.

## Docs

The reedengine and reedcli package docs, the verb's help text and the `buttonSpec.Dir` comment now say the click runs in the current anchor.
