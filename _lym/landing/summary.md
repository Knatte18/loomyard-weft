# Reed: configurable action buttons on a third status line

Every reed session gets a row of clickable action buttons, defined in config rather than code.
A click runs a shell command on the run the operator is watching: halt, cancel, open in VS Code, another terminal, a new agent, and orch distill and refresh.
Adding a button is one config entry and needs no build.

## What changes

- Status line 3 holds the action row.
  `follow` leaves line 2 and is the first entry of line 3 on every session, standalone included; line 2 keeps the session buttons, check mark and clock.
  The global `status-format` pins join the stale-build re-pin (`repinStaleBindingsLocked`), so the newest binary restores them after an older one pinned its own.
- `reed.yaml` gains `buttons`, grouped by place (`task`, `beam`, `prime`).
  An entry has `label` and `run`, plus optional `confirm` (a y/n prompt) and `window: agents` (shown only in agent windows).
  The prime takes `prime` when that list is non-empty, else `beam`.
  The key is `CategoryExecutable`: the hub base and worktree files may hold it, the local layer refuses it.
- `desktop.yaml` gains `buttons` in its machine layer (`desktop.machine.yaml`), so a machine can add buttons after reed's entries of the same place.
- `lym reed button` is a new role-audience verb under the reed subtree.
  A click on a `lymbtn<N>` range runs it; it confirms, runs the command, logs it and shows a failure on the status line.
- The shipped default set, in both the POSIX and Windows templates:
  - `task`: `HALT` (`lym tender pause {slug}`), `CANCEL` (`lym batten cancel {slug}`, confirm), `code`, `term`, `agent`.
  - `beam`: `code`, `term`, `agent`, `distill` and `refresh` (both confirm).
  - `prime`: `[]`.
- `hubgeom.ReedGeometry` hands the engine a lazy closure with the session's place, `{slug}`, `{worktree}`, `{session}`, the machine buttons and a `{beam}` resolver.
  `internal/shell` gains detached pane-shell forms (output discard, forced zero exit, command flag, quoted-word directory change).

## Design decisions

- **Render into session options.**
  tmux's status formats are one global array, so at boot and in the attach pre-flight reed renders a plain row, an agent row, the button spec and `@lym_button_dir` in one chained tmux invocation.
  A click runs the command drawn under it even after a config edit; a config edit reaches a live session at its next attach or boot.
- **Placeholders.**
  `{slug}`, `{worktree}`, `{beam}`, `{session}` and `{strand}` are filled per value; unknown brace text stays literal.
  In `run` each value is shell-quoted for reed's pane dialect; in `confirm` it is plain with `#` and `%` doubled.
  `{strand}` is read at click time from the new pane option `@lym_strand_name`, since `@strand` holds only the role.
  Path values are anchors, not worktree roots, because `lymcwd.Resolve` requires the cwd to equal `AnchorPath`.
- **Run directory.**
  A task session's commands run in its beam's anchor (the prime's when that beam is absent here), so the default CANCEL needs no `cd`; `{worktree}` still names the task.
  An entry using `{beam}` is left out with a Warn when the beam has no anchor on this machine.
- **A bad button costs only itself.**
  `LoadConfig` of reed and desktop decode `buttons` as an untyped YAML node; the render validates each group and entry and skips an invalid one with a Warn naming file, group and index.
  Labels and prompts may hold no `#`, `%` or control character, which bars tmux format, style and time-field injection.
  `{strand}` is valid only with `window: agents`.
  A `buttons` key that is not a mapping fails that config's load.
  In desktop's machine layer a group written as a mapping, an empty one included, is refused by the shape check, so desktop's load fails, adds no machine buttons and fails that load's `afterWindow` report.
- **No view mode.**
  tmux 3.6 opens view mode for a background `run-shell` that exits non-zero.
  The click line and the confirmed line discard the verb's output, including the directory change's, and end with a forced zero status, and `lym reed button` exits 0 on every path.
- **Confirm is re-checked.**
  The confirmed run re-reads the entry from the spec and refuses, with a status message, when its `run` no longer hashes as it did at the prompt.
- **Trust bound.**
  A click runs only what the session's spec holds at that index.
  A process that can write a file defining buttons, or reach the tmux socket, can change what a click runs, which adds only the caller identity to what it can already do; accepted.
- psmux on Windows: the new pins and binding are attempted unbranched and non-fatal, like the existing status pins.

## To check by hand

Not run by hand:

- Clicking each default button in a live tmux session of each place, on tmux 3.4 and 3.6: ranges drawn and clickable from `#{E:}`, `confirm-before` prompts, and no view-mode pane after a failing or refused click.
- psmux on Windows: the third status line and the click binding.
- Between an older binary's boot or attach and the newest binary's next op, `follow` can show on lines 2 and 3 and button clicks do nothing.

Findings left open by review, to look at in the diff:

- `reed-button-groups-need-open-maps` (design, blocking): reed's `buttons` groups must load as open maps; check `internal/reedengine/config.go` and its tests that a group of any shape loads and is skipped at render.
- `click-line-cd-outside-output-discard` (design, blocking): the click line's directory change must sit inside the output discard; check the composed line in `internal/reedengine/bindings.go` and the pwsh form in `internal/shell`.
