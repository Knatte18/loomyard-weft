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
- **Identify the server's starter by a pre-spawn liveness probe alone.**
  Two worktrees booting at once each hold only their own op lock, both can see no server, and both would source; the in-server claim makes exactly one win.
  A boot that sees a live server which another worktree's `down` kills before the spawn would also be misclassed as a join and leave the new server unsourced; the before-and-after `pid` and `start_time` comparison reads the server the spawn produced.
- **Pin `destroy-unattached off` only after sourcing.**
  On tmux 3.6 the detached session dies as soon as the file's `destroy-unattached on` applies, before the pin runs.
- **Set the session-local guard and the tag in their own invocation after the spawn.**
  Concurrent boots are not serialized, so another worktree's winning claim can apply the file's global `destroy-unattached on` between this boot's `new-session` and that invocation; probed on tmux 3.6, a session created by a plain `new-session -d` after such a source died at once, while one with `set destroy-unattached off` chained into the same invocation survived.
  An untagged booting session would also go uncounted by a concurrent last-worktree `down` on a `sourced` server.
- **Pin `exit-unattached off` in a separate invocation after the source.**
  On tmux 3.6 the server exits as soon as the sourcing client disconnects, so the separate pin got "server exited unexpectedly" in five runs of five; the same pin chained after `source-file` in one invocation kept the server up.
- **Fail the boot on the guard invocation's non-zero exit.**
  The source and the pins share that invocation's one exit status; probed on tmux 3.6, a file with an unknown option or an unclosed brace made it exit 1 while both pins applied, so a typo would fail every boot that starts a server.
- **Escape glob metacharacters in the path instead of refusing them.**
  Refusing is one check with a clear error; escaping must track tmux's glob rules exactly, and a miss loads nothing behind a mere warning.
- **Quote the path for tmux's command parser inside the claim.**
  The claim's `if-shell -F` branch is a command string tmux parses again; probed on tmux 3.6, an unquoted path with a space was split into two missing files and nothing loaded.
  Quoting must track the parser's rules for quotes, `;`, `#`, braces and `$` exactly, where refusing them in D2 is one check.
- **List the reed surfaces that ignore a file-created session.**
  A list leaves any surface it omits, such as the session switch or the status bar's session row, showing that session; D8 states one rule over every session read instead.
- **Kill sessions the file creates right after sourcing.**
  A concurrent worktree's new session can appear in the same window, and without a tag reed cannot tell it from a file-created one.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Opt-in key or reverted default? — Resolved, auto-pick: opt-in `tmux_conf`, default empty (D1).
- Q2 Key name and env override? — Resolved, auto-pick: `tmux_conf`, `LYX_REED_TMUX_CONF`, matching the other reed keys (D1).
- Q3 `-f <path>` at spawn or `source-file` after it? — Resolved, auto-pick: `source-file` after spawn, before the pins; probe and spawn keep `-f /dev/null` (D3).
- Q4 Which values win on conflict? — Resolved, auto-pick: reed's pins win, by ordering (D3).
- Q5 Source on joins or only at server start? — Resolved, auto-pick: only at server start (D3).
- Q6 Bad path: refuse or warn? — Resolved, auto-pick: refuse, naming key, variable and path (D2).
- Q7 Errors inside the file: refuse or warn? — Resolved, auto-pick: warn on stderr and in the log, continue; a parse error applies nothing, a run-time error applies the rest (D4).
- Q8 Guard against options that end the server or session? — Resolved, auto-pick: session-local `destroy-unattached off` chained into the spawn, `-g destroy-unattached off` and `exit-unattached off` in the claim's own invocation, and a later check that fails the boot when the session is gone or either pin reads back other than `off`; the pin set is closed (D5).
- Q9 Test hermeticity under an env-driven key? — Resolved, auto-pick: `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF` (D6).
- Q10 Windows? — Resolved, auto-pick: key parsed, never validated or sourced (D7).
- Q11 Does reed ignore an unknown session that an operator file creates on the hub server? — Resolved, auto-pick: reed tags its own sessions in the spawn invocation, and on a server with a sourced file every reed session read counts only tagged sessions (D8).
- Q12 How does a boot know it started the server, including under concurrent boots? — Resolved, auto-pick: a server `pid` and `start_time` that differ across the spawn make a candidate, and an in-server check-and-set of `@lyx_tmux_conf_claimed` picks exactly one (D3).
- Q13 A path with glob or command-parser metacharacters? — Resolved, auto-pick: refuse whitespace and `*`, `?`, `[`, `\`, `'`, `"`, `;`, `#`, `{`, `}` and `$` (D2).
