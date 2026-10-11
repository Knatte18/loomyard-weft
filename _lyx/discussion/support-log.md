# reed-tmux-conf — support log

## Interview

The session ran autonomously, so every turn below is an exploration finding followed by a self-made pick; the ledger marks each.

1. Board: `reed-tmux-conf` has a brief and no body.
   No other entry plans work on reed's server config; `reed-layout-shares`, `reed-multi-window` and `runs-overview` touch layout and windows only.
2. History: c6dc1fffc (reed-hygiene) added `-f /dev/null` to the server spawn and the capability probe on POSIX, with two reasons in `internal/reedengine/doc.go` and `probe.go`: hermetic hub servers, and a probe whose transient server must not be kept alive by a config that creates a session.
   Its commit message lists the lost prefix, bindings and `escape-time` as a known behavior change with no replacement key.
3. Reed's boot pins `remain-on-exit` and `mouse` with `-g`, the status bar formats with `-g`, and `default-shell`/`default-command` per session, in `pinBootOptionsLocked` and the geometry pins.
   Sourcing an operator file before those pins lets reed's values win without any new override logic.
4. `reed.yaml` is per worktree; `tmux` and `debug_log` already act only on the boot that starts the shared server.
   Every reed key has a `LYX_REED_*` env override.
5. `PATTERN-test-isolation`'s "Hermetic servers" section pins that reed's own servers read no `~/.tmux.conf`; tests render `reed.yaml` from the template, so an env-driven default would leak a developer's setting into tests unless the kit clears it.
6. No `~/.tmux.conf` or `~/.config/tmux/tmux.conf` exists on the explored machine.

## Rejected alternatives

- **Revert to reading `~/.tmux.conf` by default.**
  Undoes a deliberate reed-hygiene decision made one day earlier, breaks the probe's reason for `-f /dev/null`, and makes every test environment depend on the developer's home directory.
- **Default `tmux_conf` to `~/.tmux.conf`.**
  The `${env:X:-default}` form treats an empty variable as unset, so tests and operators could not turn it off without a sentinel value; opt-in avoids the sentinel.
- **Pass the operator file as `-f <path>` at server spawn.**
  tmux shows config errors in a view-mode pane on the first session, which is reed's own pane, and a config that creates sessions runs before reed's first session exists; `source-file` returns errors to reed instead.
- **Dedicated keys for `prefix`, `escape-time` and bindings.**
  Covers a guessed subset of an arbitrary config and grows a key per request.
- **Re-source the file on every boot, including joins.**
  Re-runs bindings and hooks into a live server each time a worktree boots, and fights reed's pins applied by earlier boots.
- **Make `reed.yaml` (or just this key) hub-wide.**
  Moves a whole per-worktree module for one key whose first-booter semantics already match `tmux` and `debug_log`.
- **Warn instead of refusing a missing or relative path.**
  A named file that is missing is a config error like an unresolvable `shell`, which reed already refuses; a silent warning would leave the operator wondering why their prefix is gone.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Opt-in key or reverted default? — Resolved, auto-pick: opt-in `tmux_conf`, default empty (D1).
- Q2 Key name and env override? — Resolved, auto-pick: `tmux_conf`, `LYX_REED_TMUX_CONF`, matching the other reed keys (D1).
- Q3 `-f <path>` at spawn or `source-file` after it? — Resolved, auto-pick: `source-file` after spawn, before the pins; probe and spawn keep `-f /dev/null` (D3).
- Q4 Which values win on conflict? — Resolved, auto-pick: reed's pins win, by ordering (D3).
- Q5 Source on joins or only at server start? — Resolved, auto-pick: only at server start (D3).
- Q6 Bad path: refuse or warn? — Resolved, auto-pick: refuse, naming key, variable and path (D2).
- Q7 Errors inside the file: refuse or warn? — Resolved, auto-pick: warn on stderr and in the log, continue (D4).
- Q8 Guard against options that end the server or session? — Resolved, auto-pick: pin `exit-unattached off` and `destroy-unattached off` on every server start (D5).
- Q9 Test hermeticity under an env-driven key? — Resolved, auto-pick: `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF` (D6).
- Q10 Windows? — Resolved, auto-pick: key parsed, never validated or sourced (D7).
- Q11 Does reed ignore an unknown session that an operator file creates on the hub server? — Open; handed to the plan writer under Open risks.
