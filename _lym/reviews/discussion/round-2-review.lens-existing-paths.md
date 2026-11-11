```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing path that reads or writes the state each decision touches.

**Paths traced, per decision**

- **D1, `claude.accounts` load:**
  - Live launch engines built by `shuttleengines.New` with `fabricengine.ConfigDirs` (loom, batten, orch, creel, tender): the machine layer stacks, so this **survives**.
  - Standalone burler and webster wiring, with `Dirs{Anchor: res.StateDir}`: no machine layer, so accounts are empty. That matches Scope's "standalone keeps today's behaviour", so it **survives**.
  - `RecordReaders`, built with `configengine.Dirs{}`: it can load neither the machine layer nor the env module. The readers D6 assigns to that path therefore **break** (finding 1).
- **D2, worktree record:**
  - Reed relaunch replays the stored command: covered by D2, **survives**.
  - Hibernate or pause wake through reed resume replays the stored command: covered by the same relaunch text, **survives**.
  - The session-end usage recorder (`shedadapters.UsageRecorder`, `RecordReaders`, `LocateSessions(end.Worktree = paneCwd)`): it can reach the record only by deriving it from the path it is told, because the engine's `dirs` is zero. The record's no-record fallback to the env module **breaks** there (folded into finding 1).
- **D4, `CLAUDE_CONFIG_DIR` in the launch:**
  - Reed's pane line puts the env module's exports before `launchCmd`, and a `WithEnv` export or unset inside the launch command wins, so this **survives**.
  - Reed's server scrub strips only `CLAUDECODE` and `CLAUDE_CODE_*`, so the process's `CLAUDE_CONFIG_DIR` still reaches the tmux server env. That is consistent with D6's "process's" fallback and **survives**.
  - `OpenLaunch` has no settings without a board dir; D4 and D5 already state that, so it **survives**.
- **D5, status line in `--settings`:**
  - No existing lym code renders a `statusLine`.
  - Heddle's `.claude/settings.local.json` is outranked by `--settings`.
  - **Survives.**
- **D6, readers:**
  - `CheckResume`, `AuditForks`, `OutstandingForks` and `SessionUsage` are all told the workdir or pane cwd, so they can find the record. **Survives.**
  - `ProcessLiveness`, which gets an id only, is called from `sessionread.go`, `wait.go`, creel delivery and tender. Their engines carry hub `Dirs`, but nothing tells an engine the hub's worktree list, which "directories of the hub's worktree records" needs. **Breaks** (finding 1).
  - `lym report`: it reads the committed usage records, not transcripts, so D6 misdescribes it. **Breaks** (finding 2).
  - `tools/tokencount` gets `RecordReaders()` with zero `Dirs`. **Breaks** (finding 1).
- **D7 and D8:**
  - The orch and dobby launch through the same engine `Prepare` or `OpenLaunch` path, and the usage-limit watcher reads the pane. **Survives.**

**Findings**

1. **MEDIUM**, class `design`, at D6 ("The known directories are …") and D2's no-record fallback.
   - **Summary:** D6's known-directory set (listed accounts, every hub worktree record, the env module's value) cannot be formed on the reading paths that run today. The record does not say how that input reaches them.
   - **Evidence for zero `Dirs`:** `shuttleengines.RecordReaders` builds every engine with `configengine.Dirs{}` (`internal/shuttleengines/shuttleengines.go:222-227`), stating "reading a record needs no config file". That engine serves:
     - `tools/tokencount/main.go:341`;
     - every session-end `UsageRecorder` (`internal/loomcli/wiring.go:640`, `internal/battencli/drive.go:496`, `internal/gatecli/cardfixer.go:128`).
   - **Consequence of zero `Dirs`:** such an engine can load neither `claude.accounts` (the machine layer needs Board and Anchor, per `internal/configengine/doc.go`) nor the env module.
   - **Evidence for the worktree list:** "directories of the hub's worktree records" needs the hub's worktree list. An engine is told absolute paths and derives no geometry (`PATTERN-cwd-resolution` "Told geometry"), and `ProcessLiveness(sessionID)` gets an id alone (`internal/shuttleengine/claudeengine/prober.go:30`).
   - **Fix:** the record should state how the known-directory set reaches each reader, for example a told list or `Dirs` passed to `RecordReaders`, and its bound.

2. **MEDIUM**, class `design`, at D6's third bullet and the acceptance criteria "D6: `lym report`'s cost reading counts the sessions under every known directory…" and "D6: `lym report` counts a landed run's sessions that ran on a non-default listed account…".
   - **Summary:** `lym report` reads no transcripts. It prices landed runs from the usage records committed in each run's tree (`internal/reportengine/cost.go:1-2`, "Each run's figures come from the usage records it keeps in its tree"; no `LocateSessions` call in `reportengine`).
   - **The transcript read happens elsewhere:** at session end, in `shedadapters.fillFromTranscripts`, through `LocateSessions(end.Worktree)` (`internal/shedadapters/sessionusage.go:103-112`). That path is told the pane cwd and falls back to shuttle's counts when it finds no transcript.
   - **Consequence:** D6 names the wrong reader. The two criteria test a path that does not read transcripts, and the path that decides a non-default account's cost, the session-end recorder, has no criterion.

Tool calls:
1. Burler and webster standalone wiring, `RecordReaders` callers, and statusLine uses: the manifest lacked who builds reader engines and with which `Dirs`.
2. `internal/reportengine/cost.go` head and transcript use: to confirm what `lym report` reads.
3. `internal/shedadapters/sessionusage.go` recorder and `SessionEnd.Worktree`: one hop past `RecordReaders`, to see what the session-end reader is told.
4. creel delivery and tender engine wiring: to see which `Dirs` the id-only liveness readers' engines carry.
