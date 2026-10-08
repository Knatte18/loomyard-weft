# reed-view — support log

## Interview

The session ran autonomously: no operator answered, and every open point was settled by this agent from the board entry, the related notes and the code.

1. Read the board entry `reed-view` (three bundled notes: strand and run tabs, narrow-terminal run row, a color per strand role) and the related entries `runs-overview`, `reed-multi-window`, `ide-prime-setup`, `ide-opt-in`, `operator-shortcuts`, `gate-checklist-pane` and the done task `lifecycle-waits`.
2. Found that `lifecycle-waits` (commit fd5b0f58e) added `batten:<slug>` windows to the prime's session and a strand-window seam in `windowtarget.go`.
   The board's line-1 format loops over the current window's panes, so from a batten window it would show no strands; resolved by marking the strand window (D3) and listing other windows on line 1 (D4).
3. Found that reed already sets the pane title to the full strand name with `allow-set-title off`, so the board's premise that Claude Code overwrites `pane_title` no longer holds.
   `@strand` is still needed, for the short role label rather than for protection (D2).
4. Found the existing single-line status pipeline (template, config key, `waits` token, `lyx reed statusline`, `tokenvocab`).
   The operator's bar has no place for its text; settled on retiring it and moving wait marks onto the strand buttons (D16, D4).
5. Checked on a throwaway tmux 3.6 server that `#{S:}` iterates sessions in id order, that `S/n`, `S/i`, `S/t` sort flags exist and that session user options read inside the loop.
   Read the live hub server's option defaults: tmux 3.6 ships list-mode `status-format` lines with `#{P:}` and `#{S:}` loops, and its default `MouseDown1Status` is `switch-client -t =`.
6. Read Claude Code 2.1.294's `/color` command: "Set the prompt bar color for this session", with eight colors plus `default`, and Claude Code's own mapping of those names to tmux colors.
7. Found that the orch's typing is owned by the orch watcher (`PATTERN-orch-pane-single-writer`), so shuttle cannot type `/color` into the orch (D14).
8. Found no hub-wide step that recreates sessions after a server restart: battens die with the server and each pair's session returns when its batten revives it.
   Settled on an ordered pre-boot of the hub's sessions, told to reed by the CLI layer (D12).
9. Found the operator's user `keybindings.json` already holds an unmarked hand-written copy of the five Alt entries; the marked block leaves it alone (D15).

## Rejected alternatives

- **Render line 2 as a static, Go-computed session list sorted by a stored spawn-order key**, re-rendered whenever reed creates or ends a session.
  Rejected: membership becomes a snapshot; two concurrent session creates can leave a run off the bar until the next re-render, which is the very failure this task fixes.
  The live `#{S:}` loop is always exact, and only its order after a restart needs help (D12).
- **Order sessions by name or by `S/t`.**
  Rejected: names are worktree names and do not follow spawn order; creation time cannot be set, and the operator wants spawn order.
- **Recreate sessions in order from inside the booting worktree's op lock.**
  Rejected: two worktrees booting at once would each hold their own lock and wait on the other's.
- **Leave restart order to whatever order battens revive pairs.**
  Rejected: the operator asked for spawn order to survive a restart.
- **Keep the single-line status text, shown on line 2's right side.**
  Rejected: it competes with the run tabs on exactly the narrow terminal this task targets, and its identity text repeats what the lit run tab says.
- **Keep the status-line config key and verb but stop displaying them.**
  Rejected: dead surface.
- **Set pane-border options globally.**
  Rejected: a batten window's single pane would get a `selvage` border title; window-scoped pins on the strand window avoid it.
- **One central role-to-color table.**
  Rejected: it would repeat every module's role names; the spawning module names a segment instead, and the colors stay in one table keyed by segment.
- **A color per role, or only driver against the rest.**
  Rejected: the operator preferred one color per loom segment.
- **The agent sets its own color from its stencil.**
  Rejected: lyx types skills rather than asking agents to load them (`PATTERN-role-skills-typed`); the same holds for the color.
- **Shuttle types `/color` into the orch.**
  Rejected: breaks `PATTERN-orch-pane-single-writer`.
- **Alt+Left/Right switch windows within a session** (the `runs-overview` note's assumption).
  Rejected: the operator's later 2026-10-06 decision gives them to run switching; windows are reached by click and tmux's prefix `n`/`p`.
- **`switch-client -n`/`-p` for Alt+Left/Right.**
  Rejected: they step in name order while the bar draws in id order.
- **A shell helper in the binding instead of a lyx verb.**
  Rejected by the operator on 2026-10-06; it is also shell-dependent on Windows.
- **The switch verb printing a success envelope.**
  Rejected: `run-shell` would open an output view on every key press.
- **Seeding VS Code keybindings from `lyx ide menu` or a new verb.**
  Rejected: `spawn` is the verb that writes VS Code setup today; a new verb adds surface no one asked for.
- **Removing the operator's hand-written unmarked Alt entries.**
  Rejected: lyx never edits outside its markers.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — Where do the strand buttons come from when the current window is a batten window? Auto-pick: from the window marked `@lyx_strands`, so they show from any window of the session (D3, D4). Resolved.
- Q2 — Should batten windows be reachable from the bar? Auto-pick: yes, as window buttons on line 1, only when the session has more than one window (D4). Resolved.
- Q3 — Keep `@strand` given the pane title is already protected? Auto-pick: yes, as the short role label; the title keeps the full name (D2). Resolved.
- Q4 — What happens to the single-line status text and its waits segment? Auto-pick: retire the pipeline, waits move onto the strand buttons (D16, D4). Resolved.
- Q5 — Pin at server boot only, as the board says, or also at attach? Auto-pick: both, so a running server picks them up (D6). Resolved.
- Q6 — Global or scoped pins? Auto-pick: `status-format` global only, `status` session-scoped, pane-border options window-scoped, bindings server-wide (D6). Resolved.
- Q7 — Board open item: clicking a strand button while zoomed unzooms. Auto-pick: replace `MouseDown1Status` so a strand button zooms its strand, `VIEW` unzooms (D9). Resolved.
- Q8 — Board open item: does Claude Code need Alt+arrows or Alt+z? Auto-pick: accept the loss, recorded as a risk; Ctrl+Left/Right still jump words. Resolved as an accepted cost.
- Q9 — Board open item: Alt+Up/Down cycling through Selvage. Auto-pick: a pure tmux sequence that steps over the one pane without `@strand` (D8). Resolved.
- Q10 — How does the switch verb avoid cwd resolution and `PATH` lookup? Auto-pick: every input is a flag; the binary path comes through `panebin.go`, widening `PATTERN-pane-binary-resolution` (D10, D11). Resolved.
- Q11 — Who recreates sessions in spawn order after a restart, and where does the order come from? Auto-pick: reed's boot, from an ordered list the CLI layer tells it, built from the pairs' run start times; each up runs under its own lock, never nested (D12). Resolved.
- Q12 — Narrow-terminal option for the run row. Auto-pick: option 1 (current full, others 12 characters, list mode), the note's recommendation; list mode also on line 1 (D4, D5). Resolved.
- Q13 — Board open question: one color per segment or driver against the rest? Auto-pick: per segment, the operator's preference in the note; table in D13. Resolved.
- Q14 — Board open question: does the spawning code or the agent's prompt set the color? Auto-pick: the spawning module names the segment on the launch spec; lyx types the command (D13, D14). Resolved.
- Q15 — Board open question: do the tmux border and the strand tabs carry the same color? Auto-pick: yes, both from `@strand_color` (D2, D4, D6). Resolved.
- Q16 — Which `lyx ide` verb seeds the keybindings, and where is the file? Auto-pick: `lyx ide spawn`, per-OS user directory; WSL recorded as a risk (D15). Resolved.
- Q17 — Does the orch get a color? Auto-pick: yes, the coordinator color, typed by the orch watcher (D14). Resolved.
