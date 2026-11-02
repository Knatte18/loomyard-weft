# Reed code button reports a wrong anchor path from inside the anchor directory

`reed-code-wrong-anchor` · Task · bug, reed · running · Code-Bouncer

Imported from [issue #10](https://github.com/Knatte18/loomyard/issues/10).

Seen in the Models hub, reported by the operator.

A worktree has an anchor path other than ".".
A terminal runs in the anchor directory, and the operator presses the "code" button in Reed.
Reed answers that the terminal is in the wrong anchor path, though it is in the anchor directory.

Expected: the button accepts a click whenever the worktree's anchor is where the worktree says it is.

## Investigation

Read from the source only: the bug was not reproduced, so the cause is unconfirmed.

The "code" button runs `lym ide spawn {slug}` (`internal/reedengine/template_posix.yaml`) as a child process of the click, with its cwd set to the button's stored `spec.Dir` (`internal/reedengine/buttonclick.go`).
`lym ide spawn` resolves its cwd through `lymcwd.Resolve`, whose anchor gate requires the cwd to equal `<worktree>/<anchor>` exactly (`internal/lymcwd/anchor.go`, `checkCwdAnchorGate`).
The "wrong anchor path" message therefore comes from the button's child process, judged on `spec.Dir`, and never from the terminal the operator stands in.

`spec.Dir` is frozen into the tmux session option `@lym_button_spec` at boot and attach, and re-rendered only at the next attach.
The click checks only that the directory still exists.

Candidate causes:

1. A stale spec: the anchor was set or changed (`lym fabric anchor --set`, or a reconcile writing the marker) after the session last rendered, so the old directory exists but no longer equals the anchor. This fits an anchor other than "." best.
2. On a task session the button runs in the parent's anchor, or the prime's when the parent is absent (`internal/hubgeom/buttons.go`); a parent whose recorded anchor differs from a fresh resolve of that directory fails the same gate. No code path producing that was found.

Settling observation: the click logs `reed: running a status-bar button` with `dir` and `script`, and `reed: a status-bar button ended` with the output, at Info under the session worktree's `.lym/logs`; compare the logged `dir` with the anchored directory in the error text.

## Fix

At click time do not trust the stored `spec.Dir`: run the button's command from the anchor path read afresh through `lymcwd.NamedWorktree(hub, name).AnchorPath()`, or treat a `spec.Dir` that fails the anchor check as stale and say to reattach with `lym reed attach`.
Add a test that sets the anchor marker after the render and then clicks.

## Run instructions

The cause is unconfirmed, so the writer first confirms it from the code, for example by a test that sets the anchor marker after the render and then clicks, and writes the fix for the cause it confirms.
When it cannot confirm either candidate, it stops and reports that rather than guessing a fix.
