# reed: an RC button per agent, and Slack only in the orch's session

reed's status bar showed the orch's slack and RC buttons in a task's own session.
A click there switched the parent orch, the prime's or the beam's, never the task's agent.
That happened because a task session's `@lym_button_dir` names the parent's worktree, and both state jobs resolved their orch from it.

## Slack

The Slack state line now reads `--dir #{@lym_orch_dir}`, a new session option the button-row render pins alongside the other row options.
It holds the session's own anchor for the prime and a beam, and the empty word for a task.
`slack bar` prints no slack word for an empty directory, so a task's session draws the away toggle alone.
The away toggle stays in every session, since the presence belongs to the machine.

## RC

The RC button now belongs to the agent of the window's active pane, an orch or any other agent.
Its state is a pane user option, `@lym_rc`, so there is no `#()` job:
the button is drawn while the active pane carries `@strand` and the RC click line is pinned, and it is lit while the pane carries `@lym_rc`.
The mark dies with the pane, so a relaunched agent starts unlit.

- `lym orch rc-on` reads the clicking client's active pane (`reedengine.ClientAgentPane`).
  It queues creel's remote-control command into the agent the pane runs, under the name `remote_control_name` forms for that agent, then sets the mark (`reedengine.MarkPaneRemoteControlOn`).
  A pane that runs no agent, or one already marked, is refused and nothing is sent.
  The verb still runs from the session's run directory, so a task's click passes the orch wiring through the parent's worktree.
- A launch of the orch with `remote_control` on sets the mark on the new pane through `Engine.MarkRemoteControlOn`.
  No other agent's RC is on until its button is clicked, and `orch.yaml`'s switch is unchanged.
- Removed: the orch's `remote-control.json` record (`orchengine.RemoteControlRecord` and its paths), the hidden `orch rc-bar` verb, `@lym_rc_state_command`, and the `rc-on`/`rc-off` state words.

The bar's doc comments (`bar.go`, `reedengine` and `orchcli` package docs) now describe what is drawn.

## Design decisions

- **A pane mark instead of a per-strand record file.**
  A pane option needs no new on-disk record or path, ends with the pane as the agent's session does, and the bar reads it directly.
  `@lym_rc` joins `@lym_strand_name` as a pane option a Go decision reads: the click reads it to refuse a second click.
- **No engine check on the click.**
  The pane does not show which engine an agent runs on.
  An agent on an engine with no remote control gets nothing typed for the command, and its pane is still marked.
- A `remote-control.json` left in an orch's `.lym/orch` directory is no longer read and is harmless.
  A running server keeps a stale `@lym_rc_state_command` global until it restarts; nothing reads it.

## Tests

- `reedengine`:
  - The bar, bindings, pane-binary and button-row tests pin the new RC format, the `@lym_orch_dir` pin (the quoted anchor for prime and beam, `''` for a task) and the Slack line's new `--dir`.
  - The new `remotecontrol_test.go` covers the pane read, the mark and the engine op.
- `orchcli`:
  - The launch test pins that a launch with a name marks the new strand's pane and one without marks nothing.
  - The click test covers an orch and a task agent, both refusals, an unparseable name, a failed inject and a failed mark.
  - The live-strand start test pins that adopting a running strand marks nothing.
