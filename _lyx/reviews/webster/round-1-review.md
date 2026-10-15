---
verdict: APPROVED
---

No findings.

Range: `git diff $(git merge-base main HEAD)..HEAD`, since `product.parent` is `main` and no `_lyx/loom/rework` round exists.
Unit and `integration`-tagged tests of `internal/reedengine`, `internal/reedcli`, `internal/loomcli` and `cmd/lyx` pass.

Focus directive answers:

- Follow-state read: `followingLocked` treats only the trimmed `off` as off, reads unset as on, and returns false with a Warn on a failed read.
  `ownwindow_test.go` and `attach_test.go` pin the unset, on, off and failed-read rows.
- Write and select failures: `FollowRun` logs a failed option write and a failed listing or select and returns only the no-session and state-load errors, and `loomcli/start.go` logs a `FollowRun` error and goes on.
  `FollowSession` returns its own wrapped errors, which `reedcli/follow.go` puts in the error envelope.
- `followTargetWindow` against `relevantWindow`: both pick the highest-index live own window, else the overview.
  The one difference, an unmarked overview from an older lyx, yields the same window, as `doc.go` states.
- Start entries: the single `FollowRun` call in `startCmd` sits after `Up` and `ensureStatusStrand`, so every start path reaches it.
  `arm.go`'s `run` and `step` entries and `resume` do not call it.
- Bindings and button: Alt+Up, Alt+Down and the shell, pane and window clicks write off first; the session click, Alt+Left and Alt+Right do not.
  The button writes off from on and, from off, writes on and runs the pinned jump line, which `bindings_integration_test.go` drives against real tmux.
  The jump line follows the `composeSwitchCommand` quoting.
- Test economy: the two added top-level tests match the `test-budget.yaml` bump of two.
  The engine, cli and loom layers each pin their own layer's behavior, and I found no duplicated row.
- Docs: `reedengine/doc.go`, `reedcli/doc.go`, `loomcli/doc.go` and `PATTERN-pane-binary-resolution.md` describe follow, the verb and the follow button.
