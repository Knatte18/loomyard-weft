```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each decision, trace the code that already writes or reads the same state, and say whether the decision's result survives it.

**Findings**

1. **BLOCKING — D10 (with D6). The account launch and reed's env scrub undo the area's `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR`.**
   - `resolveLaunchAccount` in `internal/shuttleengine/claudeengine/accountenv.go` writes `CLAUDE_CONFIG_DIR` on every launch. It exports a listed account's own config directory, or unsets the variable for an account on the default `~/.claude`. With no account, the session reads the env module's `CLAUDE_CONFIG_DIR`, which is hub config an agent can write.
   - `claudeengine.ScrubbedEnv` (`scrub.go`) declares `CLAUDE_CONFIG_DIR` and the `CLAUDE_CODE_` prefix as scrubbed. `shuttleengines` hands that scrub to reed, which strips those keys before the tmux server spawns (`lifecycle.go`, `e.scrub.Apply(os.Environ())`).
   - So a variable set by `lym area exec` or in the server's environment never reaches a pane. An account launch then points the session at the host's `~/.claude` (denied, so the launch fails) or at an account directory outside the area.
   - The record says nothing about the account rotation, the env module key or the scrub. It needs a decision: inside an area, the account resolution and the launch line take the area's config directory (or per-account directories under it). The env module's `CLAUDE_CONFIG_DIR` is ignored inside an area. `CLAUDE_CODE_TMPDIR` is set by the launch line, not inherited.

2. **BLOCKING — D6 (with D8). Finding another hub's creel socket reads the caller's own temp root, and inside an area the caller cannot see which hubs have an area.**
   - `hubgeom.CreelGeometry(hubPath, tempRoot)` builds `SocketPath` as `tempRoot/SocketName(...)`, and the caller passes its own lymtemp root.
   - Inside an area, `TMPDIR` points at the area root, so the in-area daemon's outbound worker and `confirm` would look for a non-area hub's socket under the area root. A non-area hub's daemon would look for an area hub's socket under `/tmp/lym-<uid>`.
   - D6 says `lymtemp` and `hubgeom` derive the root "from the hub path and the area file's presence". But D6's ruleset grants no read on `~/.config/lym` beyond `machine.json`, so inside an area that presence check cannot be made for any hub, its own included.
   - Cross-hub delivery breaks in both directions as written. The temp root must be derived per target hub, from a source readable inside the area, such as the hub record written by `lym area start`. `CreelGeometry` must stop taking the caller's root for another hub.

3. **MEDIUM — D7 (with D3). The host-side boot reads hub config that D7 says area verbs never read.**
   - D3 routes reed's server spawn through `lym area start`, and a host `lym reed attach` boots through it.
   - reed's preflight and spawn read the hub's reed config: the `tmux` executable, `shell` and the sourced `tmux_conf` (`preflightBootHost` in `lifecycle.go`).
   - The record does not say whether `lym area start` builds the tmux command line from hub config read on the host (which contradicts D7's "tmux from `PATH`") or from defaults.
   - The tmux binary runs only after `exec` under the restriction, so no host escape follows. The record should still state which source the boot uses and that `tmux_conf` and `shell` take effect only inside.

4. **MEDIUM — D10 / D6. The user-setup source would read the host's `~/.claude`, which D6 does not grant.**
   - `UserSetupSource.resolveConfigDir` in `claudeengine/usersetup.go` uses an injected directory or falls back to the default `~/.claude`.
   - It reads instruction files, installed plugin roots and skills from there.
   - D6 grants no read on `~/.claude`; D10 grants the plugin cache only.
   - D10's general sentence ("every lym reader … follows the variable") covers this only if the resolution reads the area's directory. Given finding 1, today it would resolve the host default and fail. So this is the same gap as finding 1, from the reader's side.

5. **LOW — D8. The hub record survives.**
   - `hubregistry.Record`'s only caller is `creelcli/hubdirectory.go`, at daemon start. Skipping it inside the area and writing it from `lym area start` covers every writer.
   - Other hubs' daemons prune only entries whose path is gone, so the area hub's entry survives.

**Paths traced, per decision**

- D3: reed's preflight socket-length check reads `TMUX_TMPDIR`, which D6 sets. It survives: the area root path is short.
- D3: reed's server env scrub breaks D10, as in finding 1.
- D6: `CreelGeometry`'s temp root breaks, as in finding 2.
- D6: lymtemp's "a run directory's root is taken as its own" survives.
- D7: reed's boot reading hub config is unstated, as in finding 3.
- D8: `hubregistry.Record` and the prune by other daemons survive.
- D8: `peer_linux.go`'s `SO_PEERCRED` is changed by D8 itself, which the record states.
- D9: the reed watchdog's OOM read survives. It reads the cgroup under `/sys`, which D6 grants, and the scope keeps its `lym-pane-` name.
- D9: the journal read degrades to a count, which the record states.
- D10: `accountenv`'s launch line and the account rotation break it (finding 1).
- D10: the env module's `CLAUDE_CONFIG_DIR` breaks it (finding 1).
- D10: `usersetup`'s config-directory resolution breaks it (finding 4).
- D10: `claudeengine.ScrubbedEnv` breaks it (finding 1).
- D10: `codexengine` has no `CODEX_HOME` handling today, so the new variable has nothing to collide with. It survives.

**Tool calls**

- Grep for readers of the Claude home directory and callers of `hubregistry.Record`: my context lacked who else writes the config directory and the hub record.
- Read `accountenv` and grep `usersetup`, heddle, `reedengine` and `codexengine` for config-directory and env handling: my context lacked how launches set `CLAUDE_CONFIG_DIR`.
- Read `accountenv`'s account resolution and reed's `preflightBootHost` and scrub: my context lacked the scrub and the host boot's config reads.
- Read `envscrub` and grepped creel's socket-path functions: my context lacked the scrub's key sources and where creel's socket path is formed.
- Read `shuttleengines`' scrub wiring and the `SocketName` callers: my context lacked which keys are scrubbed and who builds the socket path.
- Read `hubgeom/creelgeom.go` and `claudeengine`'s scrubbed env: my context lacked `CreelGeometry`'s temp-root source and the Claude scrub list.
