# Support log — reed-add-parentless-placement

## Interview

### Turn 1 — exploration, autonomous

No operator answers in this session; every question was self-picked and is in the ledger.
Read the board entry (#573, issue #673) and the reed-related notes `reed-status-buttons` and `reed-status-buttons-prototype`; neither plans work on strand placement, and both build on the window buttons this task keeps.
Explored `internal/reedcli/add.go`, `internal/reedengine/render`, `internal/reedengine/doc.go`, `selvagepane.go`, `apply.go`, `reapply.go`, `watchloop.go`, `namerepair.go`, `internal/reedcli/watchdog.go`, `internal/shuttleengine/spec.go`, `internal/shuttlecli/run.go`, the reed config templates, `tools/sandbox/SANDBOX-REED-SUITE.md` and `docs/overview.md`.
Findings: `lyx reed add` alone defaults to `below-parent`; shuttle defaults to `own-window`; orch, loom and seats name `own-window` or `overview`; no production spawner names `below-parent` or `top`; no production caller reports content rows; the resize watch loop re-applies only the shell-window layout; the per-hub daemon also runs name repair, and its `watchdog: off` check skips both loops.
The stencils under `contracts/stencils` and `plugins/` carry no pane-era placement wording.

### Turn 2 — design settled

Picks Q1–Q7 are in the ledger with their rationale.

### Lens review

- premises, MEDIUM, D3 relaunch does not kill a non-`@lyx_own` old pane and places the new window after the shell: fixed, D3 now moves a leftover pane into its own window with `break-pane` first.
- premises, MEDIUM, D7 re-pin never unbinds `M-z`: fixed, explicit `unbind-key` in D7 and an acceptance row.
- premises, LOW, `watchdog` key gates only the resize loop: rejected in part, `reedcli/watchdog.go`'s "off" check skips name repair too; D6 now keeps the key gating name repair, firmly.
- premises, LOW, the existing hook unpin clears the whole array: fixed, D6 uses the existing unconditional unpin and its bound changed.
- goal-scope, MEDIUM, D6 left the key conditional: fixed, merged with the premises finding above.
- goal-scope, MEDIUM, the grep criterion contradicts D2's and D3's own strings: fixed, refusal and remap added as exceptions.
- goal-scope, LOW, stencil sweep not recorded in the record: fixed, one line in Scope.
- goal-scope, LOW, old daemon also exits on binary replace: fixed, the daemon risk is dropped since it ends at the replace.
- goal-scope, LOW, the `selvage` block empties: fixed, the block retires whole in D5.
- goal-scope, LOW, `--parent` help: fixed, Scope Out says it is already accurate.
- board-fit, MEDIUM, `spawn-audit-log` plans a sampler on the daemon: fixed, named in Scope Out.
- board-fit, LOW, the overview's reed module-table row: fixed, named in D8.
- board-fit, LOW, status-button notes fit: no change needed.
- pattern-fit, MEDIUM, `SANDBOX-REED-WATCH-SUITE.md` missed: fixed, named in D8.
- pattern-fit, LOW, PATTERN-leaf-packages does not list render: fixed, the constraint cites render's package doc.
- pattern-fit, LOW, size keys are kept on rewrite, not removed: fixed, D3 and the constraint say so.
- scenarios, MEDIUM, relaunch leaves the old pane alive: fixed by D3's move and an acceptance row.
- scenarios, MEDIUM, a dead leftover pane: fixed, D3 moves dead panes the same way.
- scenarios, MEDIUM, Selvage geometry after boot and heal: fixed, D4 requires Selvage alone filling the window after boot, heal and move, with an acceptance row.
- scenarios, MEDIUM, the hook match by command string misses other binaries: fixed by D6's unconditional unpin.
- scenarios, LOW, older long-lived processes re-pin the hook: fixed, now the one Open risk.
- scenarios, LOW, remove of a remapped strand moves a shell viewer: fixed, the move makes it an ordinary own-window strand; acceptance row added.
- scenarios, LOW, the strands'-window resolver fallback: fixed, a plan-writer note.
- scenarios, LOW, invalid `--anchor` values and `--parent` with the default: fixed, acceptance rows added.
- leanness, MEDIUM, Scope In restates the Decisions: fixed, Scope In points at the Decisions.
- leanness, MEDIUM, Constraints padding: fixed, general rules cut.
- leanness, LOW, D1 enumerates spawners: fixed, moved to Interview turn 1.
- leanness, LOW, D2's empty bound: fixed, cut.
- leanness, LOW, Auto-mode assumptions repeat the Decisions: fixed, now one pointer line.
- leanness, LOW, Open risks were bounds: fixed, cut.
- leanness, LOW, Notes repeat other rules: fixed, the benchmark and test-budget bullets cut.
- leanness, LOW, Interview turn 2 repeats the ledger: fixed.
- leanness, LOW, D8 repeats Scope: fixed by trimming Scope.

## Rejected alternatives

- Change only `lyx reed add`'s default to `own-window`: leaves `--anchor below-parent` able to split a strand into the shell window, and leaves the layout, config and bindings the entry's sweep names.
- Remap a retired `--anchor` value to `own-window` silently: hides a caller's stale assumption; nothing in production passes one, so a refusal costs no caller.
- Leave an older binary's shell-window pane in place until it relaunches: the relaunch's old-pane kill, window placement, remove and viewer moves each assume an `@lyx_own` window, so each would need a special case; the lens review showed the relaunch leaves a second copy.
- Kill and relaunch a leftover pane at upgrade: ends a live agent session the operator may be in.
- Retire the per-hub watchdog daemon whole: it also carries name repair, and `spawn-audit-log` plans a sampler on it.
- Retire the `watchdog` key: its "off" spelling also turns name repair off, which a retirement would change silently.
- Remove only reed's own `window-resized` entry by command string: misses entries other binaries pinned; the existing clear-all is simpler and only reed pins that hook.
- Keep Alt+z for a shared group window: today it does nothing outside the shell window, and nobody asked for group-window zoom.
- Keep `top` as an engine-only anchor: no spawner uses it, and it places a strand in the shell window.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): retire the whole shell-window strand layout, not only the default, because the entry's sweep names layout code and the shell window must hold Selvage alone (D1, D4).
- Q2 (resolved, auto-pick): refuse retired and unknown `--anchor` values, naming `own-window` (D2).
- Q3 (resolved, auto-pick, revised after the lens review): move a leftover shell-window strand pane into its own window with `break-pane` at the first op, instead of leaving it in place (D3).
- Q4 (resolved, auto-pick): retire the resize self-heal and its hook; the daemon stays for name repair (D6).
- Q5 (resolved, auto-pick): retire Alt+z, the strand-pane buttons, the strand click and the zoom-keeping wrapper (D7).
- Q6 (resolved, auto-pick): retire `collapsed_rows`, `min_full_rows` and the `selvage` block through configengine's `RetiredKeys` (D5).
- Q7 (resolved, auto-pick): keep the `watchdog` key gating name repair (D6).
