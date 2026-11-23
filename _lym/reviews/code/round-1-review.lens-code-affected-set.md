```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked what the diff forces outside its own lines; three members are left wrong.

**Per change, the set found:**

1. **New config key `pane_memory_max_mib`** (`Config.PaneMemoryMaxMiB`, `ConfigSpec` categories, both templates): readers are `paneMemoryMax`/`capPaneMemoryLocked`, `TestLoadConfig_TemplateDefaultsResolve`, the classify test `TestModules_EveryTemplateKeyClassified` and the reconcile note in each template comment.
   - Covered: the categories table, both templates and the new assertion.
   - No user-facing doc lists reed keys outside the templates; the grep found only `docs/research`, which has no rule weight.
   - Left wrong: the `//testtiming:keep` reason on `TestLoadConfig_TemplateDefaultsResolve` (F3).
2. **New `proc` exports** (`CapProcessMemory`, `ProcessOOMKills`, `OOMKillsInCgroup`, `OOMKill`): the set is `proc/doc.go`, `export_linux_test.go`, the Windows stubs, the `cmd/lym` spawn-observability allowlist, and PATTERN-child-processes (spawn logging).
   - Covered: `doc.go` is updated, the stubs are present, and the allowlist entry is added.
   - The allowlist's import-cycle reason holds: `internal/logger/retention.go:17` imports `internal/proc`.
   - Left wrong: the busctl spawn's log level (F2).
3. **`launchStrandLocked` now caps every strand pane**: the set is its callers (`lifecycle.go:882`, `hold.go:113`, `strand.go:423/458/617`), all through the one chokepoint, and `reedengine/doc.go`.
   - Covered: `doc.go` gains its paragraph.
4. **Watchdog OOM step** (`watchedSession` fields, `reportPaneOOMKills`, step 7 in the `runWatchdogLoop` doc): the set is `reedcli/doc.go`, the `runWatchdogLoop` doc, the `cmd/lym/daemonwakeups_test.go` list, PATTERN-daemon-wakeups, PATTERN-reed-panes Layering, and PATTERN-agent-addressing (messaging goes through creel).
   - Covered: both docs are updated, and the step adds no ticker.
   - The step posts through creel via `orchcli.NotifyAsker`, which agent-addressing allows.
   - Left wrong: PATTERN-reed-panes "Reed never imports creel" (F1).
5. **`creelcli` smoke helpers** move from the `lym reed` CLI to engine calls. This is forced by the reedcli→orchcli→creelcli cycle (commit `fa1c3aa80`).
   - Covered: the CLI's `add` output is still exercised by `reedcli`'s own tests.
   - The helper's GUID check moves into `addProviderStrand`'s engine error path.
   - No assertion of a test under test is loosened.
6. **Commit messages**: `c56381f9f` carries a `Root-Cause:` trailer naming the uncapped tmux scope, which is the defect removed. There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed apart from F1.
7. **Declared PATTERN changes**: the directive declares none, so this produces no finding.

**Findings:**

- **F1 — MEDIUM, design.** Location: `internal/reedcli/oomnotice.go:11` (import of `internal/orchcli`).
  - Summary: reedcli now reaches creel (`creelcli`, `creelengine`, `creelconfig`) through `orchcli`. That breaks PATTERN-reed-panes's Layering rule "Reed never imports creel", and no `Pattern-Departure:` trailer or pattern edit records it.
  - Evidence: `pattern/PATTERN-reed-panes.md:212-213`; `go list -deps ./internal/reedcli` lists `internal/creelcli` and `internal/creelengine`. The import cycle that commit `fa1c3aa80` had to break in `internal/creelcli/smoke_helpers_test.go` confirms the new edge.
  - Why MEDIUM and not BLOCKING: the rule's own section is about creel's layering against reedengine, and the edge is transitive through `orchcli`, not a direct import. If the handler reads "Reed" as the whole reed module, this is BLOCKING.
- **F2 — LOW, consistency.** Location: `internal/reedengine/panememory.go:46` (diff line 134).
  - Summary: the busctl spawn on each pane launch is a lifecycle spawn, but it is logged at Debug. PATTERN-child-processes sets `Info` for a lifecycle spawn and `Debug` only inside a polling probe.
  - Evidence: `pattern/PATTERN-child-processes.md` "The level is `Info` for a lifecycle spawn and `Debug` for a spawn inside a polling probe"; `panememory.go` `logger.Debug("reed: moving the pane's shell into a memory-capped scope through busctl", …)`. The outcome is logged at Info or Warn afterwards.
- **F3 — NIT, consistency.** Location: `internal/reedengine/config_test.go:40`.
  - Summary: the `//testtiming:keep` reason claims to pin "every shipped template default" and lists them, but the list leaves out the new `pane_memory_max_mib 8192` the test now asserts.
  - Evidence: `config_test.go:40` (the reason text) and the diff's added assertion at `config_test.go:76-78`.

Tool calls:
1. Grep for reed config key docs, the logger→proc import, agentname program rules, and reed/creel import rules. My context lacked any doc listing, the allowlist reason's premise, and where the layering rule's text sits.
2. Read the `config_test.go` keep marker, the PATTERN-reed-panes Layering text, and the daemon-wakeups scan list. My context lacked the keep reason text and the scan list.
