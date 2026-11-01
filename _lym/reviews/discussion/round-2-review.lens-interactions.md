```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked pairs of decisions that act on the same state, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D5 (`status-format[2]` global pin against the per-session row options rendered at the pins): held. A session booted by an older binary shows `follow` alone until its next attach, and D1's Bound states this.
- D1 × D7 (stale-build re-pin against the click binding): held. An older binary's `MouseDown1Status` binding has no `lymbtn*` branch, and D1's Bound states that clicks do nothing.
- D1 stale re-pin × session `status 3`: held. The re-pin is global only, and D1 states that a session an older binary attached keeps `status 2`.
- D2 × D3 (validation at render for both sources, and `desktophook.LoadConfig` shared with `afterWindow`): held as stated. D2 concedes that a group written as a mapping fails the desktop load, so `afterWindow` fails too.
- D2 × D3 (prime fallback, applied to reed's groups and the machine's independently): held. D3 states the composition.
- D2 bullet "a group written as a mapping fails that config's load" × D2 render-skip and its acceptance criterion: broken for reed's layers (F1).
- D2 `{strand}`-only-with-`window: agents` × D5's two rows: held.
- D3 × D4 (failed desktop load inside the closure): held. Only the machine buttons drop; the place and values stay.
- D4 × D7 (`{beam}` absent → its entries left out; run directory falls back to the prime's anchor): held.
- D4 read failure × D7 run directory: held. Only `follow` shows, so no run directory is used.
- D5 × D7 (spec run directory and `@lym_button_dir` written in one chained invocation; `run-shell -c` against the verb's own presence check): held.
- D5 × D6 (row chosen by `@strand`, `{strand}` read from `@lym_strand_name`): held. A pane spawned by an older binary lacks the new option, and D7 shows a message and runs nothing.
- D6 × D7 confirm path (hash over the spec `run` with `{strand}` unfilled, plus a grammar-checked strand name; `#` and `%` barred or doubled in `confirm`): held.
- D7 run directory "no button process holds a task worktree as its cwd" × D8 `agent` entry: broken (F3).
- D7 rationale "CANCEL removes the task worktree" × D8 CANCEL entry `lym batten cancel {slug}`: broken premise (F2).
- D7 exit 0 × D8 CANCEL ending the session (client gone): held. The failed `display-message` is logged at Debug and the verb still exits 0.
- D7 × D8 HALT and `term` from the beam's anchor: held.
- D2 `CategoryExecutable` × D3 `CategoryMachine` for a key that runs commands: held. `window_command` is the precedent for a Machine key that runs a command.
- Acceptance criteria D1, D3 to D8 against their decisions: held. D2's criterion diverges from D2's own bullet (F1).
- Constraints (tmux 3.4 floor, psmux, `PATTERN-spawn-observability` against D7's Info log, `PATTERN-config-layers`): held.

**Findings**

- **F1 — MEDIUM, design, D2 (third config bullet) and its acceptance criterion.**
  - **Summary:** D2 says a group written as a mapping fails the config's load. For reed's base and worktree layers the load admits it, so the bullet contradicts D2's own render-skip path and acceptance criterion.
  - **Evidence:** `internal/yamlengine/fill.go:111-123` checks only template mapping against file non-mapping, and template scalar against file mapping; a sequence template node has no check. `internal/yamlengine/reconcile.go:177` skips list-element paths when the list key is present. `internal/configengine/config.go` `warnUnknownKeys` only warns. Only `ValidateMachine` refuses, at `internal/configengine/layers.go:319-337`, because the mapping's leaf is not in the template. So the bullet holds for `desktop.machine.yaml` but is false for `reed.yaml`.
  - **Effect:** a plan written from the bullet expects a load failure that never comes, while the render-skip path must still handle the mapping group.

- **F2 — LOW, design, D7 run-directory rationale ("CANCEL removes the task worktree").**
  - **Summary:** D8's CANCEL runs `lym batten cancel {slug}` without `--remove`, and that keeps the pair. The Windows cwd premise D7 gives for the run directory is therefore false.
  - **Evidence:** `internal/battencli/cancelcmd.go:56` says "The pair is kept by default"; removal needs `--remove`.
  - **Effect:** the run-directory choice still stands, because `ResolveBattenPrime` requires the prime or a beam (`internal/battencli/refusal.go:57-78`), as D8's rationale says. Only D7's stated reason is wrong.

- **F3 — LOW, consistency, D7 ("no button process holds a task worktree as its cwd") × D8 `agent` entry.**
  - **Summary:** In a task session, D8's `agent` runs `cd {worktree} &amp;&amp; lym shuttle open …`, so the button's shell holds the task worktree as its cwd for the whole of `shuttle open`. That contradicts D7's absolute claim.
  - **Evidence:** decision-record.md D8, `agent` entry, and D7 run-directory bullet.

Tool calls: none; the inherited context held everything this lens needed.
