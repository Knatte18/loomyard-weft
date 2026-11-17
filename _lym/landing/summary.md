# reed: an RC button per agent, and Slack only in the orch's session

Reed's status bar showed the Slack and RC buttons in a task's tmux session as well as in an orch's.
A task session's `@lym_button_dir` names its parent's worktree, so a click there switched the parent orch's Slack or remote control, never the task's.
The `internal/reedengine/bar.go` comment said the Slack toggle was "drawn only in a prime's or a beam's session", which the code did not hold.

## What changes

- **Slack is per orch.**
  The Slack toggle reads its state through the new session option `@lym_orch_dir`, which names the orch's directory.
  A task session holds it empty, so its job names no orch and the bar draws the away toggle alone.
- **RC is per agent.**
  The RC button belongs to the agent of the window's active pane and is drawn only while that pane carries `@lym_strand_name`.
  A click runs the new hidden verb `lym orch rc-on` from that session's run directory, which queues creel's remote-control command into that agent's strand.
- **The RC state is a pane mark.**
  `@lym_rc` on the agent's pane lights the button and makes a second click on a marked pane a refusal.
  A launch with remote control also sets the mark.
  Every agent starts unmarked, so RC stays off until its button is clicked.
  `orch.yaml`'s `remote_control` for orch sessions is unchanged.
- **The per-orch RC record is gone.**
  The `rc-bar` verb, the `RemoteControlPath` and `RemoteControlLockPath` state paths and the record they guarded in `orchengine` are removed, since the pane mark replaces them.
  Every `lym orch` verb now needs a repository.
- `remote_control_name` forms the name for the clicked agent's strand at click time, and for the orch's at launch.
- The `reedengine` and `orchcli` docs and the bar's doc comment match what is drawn.

## Design decisions

- The state lives on the pane, a display option, not in a file, so it dies with the pane and needs no lock.
  `@lym_strand_name` and `@lym_rc` are the only pane options a Go decision reads.
- The RC click line rides its own pinned option, `@lym_rc_click_command`, set and unset apart from the Slack one.

## To check

- **`rc-button-drawn-without-button-dir`** (design, medium): the RC button needs the session to carry `@lym_button_dir`, and the bar may still draw it without that option.
  Check `internal/reedengine/bar.go` and `buttonrow.go` for a drawn RC button whose click has no directory to run from.
- Not run by hand: a real tmux click on RC in a task session, a prime session and a beam session, and a Slack click in each.
