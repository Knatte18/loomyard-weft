# Card 1 — tmux-conf-config-key

**Edit:**
- `internal/reedengine#Config`
- `internal/reedengine#ConfigTemplate`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/reedengine_test#TestLoadConfig_EnvOverride`
- `internal/testkit/tmuxkit#setEnv`
- `internal/testkit/tmuxkit#TestSetEnv`

**Intent:** Add the opt-in `tmux_conf` key (D1) and close the environment leak it opens into tagged test packages (D6), with no boot behavior change yet.

`Config` gains `TmuxConf string` with the yaml tag `tmux_conf`, placed after `Watchdog`.
Both templates gain the line `tmux_conf: ${env:LYX_REED_TMUX_CONF:-}` after the `watchdog` line.
The POSIX comment says: a tmux config file reed sources into a hub server on the boot that starts that server, before reed's own option pins, so reed's values win; empty, the default, sources nothing and keeps the hermetic server; a leading `~/` expands to the home directory, and the value must be an absolute path to an existing regular file holding no whitespace and none of `* ? [ \ ' " ; # { } $`; a changed value takes effect at the hub server's next start; an already-materialized reed.yaml lacking the key takes the default until "lyx config reconcile" writes it; set `LYX_REED_TMUX_CONF` in the shell profile instead of committing a personal path.
The Windows comment says the key is parsed but neither validated nor sourced, since psmux reads its own config at start (D7).
`ConfigTemplate`'s doc comment stops pinning a count of `${env:...}` keys and names the set instead: the two tool paths, `debug_log`, `mouse`, `watchdog` and `tmux_conf`.

`TestLoadConfig_TemplateDefaultsResolve` asserts `TmuxConf` is empty; the package runs under `tmuxkit.Main`, which this card makes unset the variable, so a developer's own value cannot reach the assertion.
`TestLoadConfig_EnvOverride` sets `LYX_REED_TMUX_CONF` to a path through `t.Setenv` and asserts `TmuxConf` carries it verbatim, since loading applies no validation.
The criterion that "lyx config reconcile" adds the key to a reed.yaml lacking it needs no new row: `TestFill`'s `reed dropped top-level key` row pins, for an `${env:...}` key, that the load takes the template default and the fill writes the key back, and the fill is key-agnostic, so the `TestLoadConfig_EnvOverride` assertion that the template carries `tmux_conf` under `LYX_REED_TMUX_CONF` completes it.

`setEnv` additionally runs `os.Unsetenv("LYX_REED_TMUX_CONF")`, beside the `TMUX` and `TMUX_PANE` unsets, so every reed.yaml a tagged test renders from the template resolves `tmux_conf` to empty.
The kit's package doc comment gains one sentence saying so.
`TestSetEnv` sets the variable through `t.Setenv` before calling `setEnv` and asserts it is unset afterwards, in the loop that already checks `TMUX` and `TMUX_PANE`.

Bound: this adds the edge through which arbitrary operator tmux commands will reach a reed server; cards 2 and 3 bound what the value may name, when it is sourced and what it can override.

**ImpactSummary:** One new Config field nothing reads yet, one template line per GOOS, one kit env unset; every existing reed.yaml keeps loading, and tagged tests see an empty key.
