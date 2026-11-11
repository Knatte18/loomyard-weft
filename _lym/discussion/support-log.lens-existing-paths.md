```yaml
lens: existing-paths
outcome: reported
```

## Findings

- **MEDIUM — D6: id-only liveness reader has no worktree to resolve the account.** D6 says the prober takes "the directory of the worktree's recorded account", but its entry point has none: `Claude.ProcessLiveness(sessionID string)` (`internal/shuttleengine/claudeengine/prober.go:30`), called from `internal/shuttleengine/wait.go:1617` and `sessionread.go:150` with only a session id. The record does not say how such a reader finds the account (via the engine's told dirs `c.dirs`, or by scanning every configured account's `sessions/` registry — session ids are UUIDs, so a scan is unambiguous). Unaddressed, a session on a non-default account reads as not alive, and `wait.go`'s death classification and attach facts misread it. `CheckResume(sessionID, workdir)` (`resume.go`) does get `workdir`, so it can resolve the account, but it builds the registry path as `home/.claude/sessions` as well as the project dir via `claudeProjectDirFor` — both must move. The decision should name the resolution route for id-only readers.
- **MEDIUM — D9/Open risks: deployed plugins go stale on the second account.** `tools/deploy/main.go:230-247` syncs lym's plugin cache only into `<home>/.claude/plugins/cache/...`. D9 has the operator install plugins into the second dir once; every later deploy updates only `~/.claude`, so sessions on the second account keep running stale skills/stencil-bound plugins with no signal. Either have deploy sync every configured account's dir or state this and give the operator a way forward.
- **LOW — D5: precedence claim names the wrong layer.** D5 says the status line wins because "the worktree's project settings take precedence over user settings". The engine renders per-session settings passed by `--settings` (`buildSettings`, `settings.go:434`); `settings.go:6` says shared settings come from the worktree's `.claude/settings.local.json`, which heddle's `claude-settings` target renders (`internal/heddleengine/target.go:24`). If the statusLine goes in the `--settings` file, it also wins over a heddle-rendered `statusLine`; if it goes in the local file, heddle's re-render could overwrite it. Name the `--settings` file as the carrier.

## Paths traced, per decision

- D1: claude module already loads via `loadConfigOrRefuse(c.dirs)` (`claudeengine.go:122`) — a tracked-layer refusal fits the existing refusing load. Survives.
- D2: reed relaunch replays stored `Cmd`/`ResumeCmd` (`reedengine/strand.go:411-423, 609-619`, `lifecycle.go:874`) — survives. Anchor move carries `.lym` (`fabricengine/anchormove.go:65-80`, `anchorchange.go:197`) — record survives. No cleanup found that removes a worktree's `.lym` subdir.
- D3: counter/usage files are new state; no existing writer touches them. Survives.
- D4: reed server scrub strips only `CLAUDECODE`/`CLAUDE_CODE_*` (`scrub.go:8-9`), not `CLAUDE_CONFIG_DIR` — an operator-shell value inherited by the reed server is overridden by the engine's export; with no accounts it stands as today. Survives. Engine env file (`command.go:309`) is fixed content, untouched. Survives. `env` module values still reach gate tiers and Drift (`gatecli`, `driftengine`) — no Claude session there, no misread. Survives.
- D5: hooks use `transcript_path`, unaffected. Survives. Heddle settings path: survives only with carrier named (LOW finding).
- D6: `projectsRoot` (`transcriptread.go:26`), `claudeProjectDirFor` (`audit.go:103`), `scope.go:144` are on the engine, which holds `c.dirs` — can resolve the account if the plan threads it. Survives. `ProcessLiveness` — breaks unless the record names its route (MEDIUM). Agent-run `lym` verbs inside a session inherit `CLAUDE_CONFIG_DIR` from the pane, so D6's `CLAUDE_CONFIG_DIR` fallback covers them. Survives.
- D7: orch and dobby launch through the same `Runner` → `Prepare` → reed, so they get the export. Survives.
- D8: usage-limit wait never relaunches with a new command; stored `ResumeCmd` keeps the export. Survives.
- D9: breaks re deploy-synced plugins (MEDIUM).

Tool calls:
1. Grep `scrub.go`, reed `Cmd`/`ResumeCmd` storage, `envFileContentFor` — my context lacked how reed relaunches and what the server scrubs.
2. Grep every `.claude` home reader in `internal`, `cmd`, `tools` — my context listed only the engine's readers, not those outside.
3. Grep `.claude` uses in reed lifecycle, settings, heddle target, deploy — to see who writes settings and plugin caches.
4. Grep `ProcessLiveness` callers and read `CheckResume` — to learn what context the liveness and resume readers get.
5. Grep `.lym` move/remove code and the engine's `c.dirs` — to check D2's record survives anchor moves and what the engine knows.
