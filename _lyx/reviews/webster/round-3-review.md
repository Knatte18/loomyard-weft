---
verdict: APPROVED
---

Round 3 settles the two focus questions and finds no defect.

## Rejected

- Session-option targets: every `set-option` and `display-message` that reads or writes `@lyx_follow` addresses the session as `=name:` (`exactSessionWindowTarget`) in `internal/reedengine/follow.go` and the boot argv in `internal/reedengine/lifecycle.go`, or as the told plain target in `follow_integration_test.go`.
  The tests that write the option (`bindings_integration_test.go`, `smoke_drivingsurface_test.go`) use `"="+name+":"`, and `follow_test.go` pins `"=" + session + ":"` for `set-option` and `"=" + session` for `list-panes -s`, matching the code.
  The remaining bare `"=" + name` targets are on `list-panes -s`, `list-clients` and `attach-session`, which take a session target.
- Resume after a server restart: `TestOwnWindow_RealTmux/ResumeAfterAServerRestartReopensDetached` now expects the viewer on the highest-index own-window strand's window, which is what follow-on relaunch produces, and passes.
- Verification: `lyx gate test --tags tmux` passes for `internal/reedengine`, `internal/loomcli` and `internal/reedcli`, covering every failing test the Publish failure section names.
