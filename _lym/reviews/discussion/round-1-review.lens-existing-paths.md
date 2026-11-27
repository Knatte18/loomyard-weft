```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**

- D1: `termwindow.resolve` already requires DISPLAY/WAYLAND_DISPLAY, and the reed tmux server keeps the starter's env, scrubbing only agent identity (`internal/reedengine/lifecycle.go:461-469`), so KDE vars reach batten's Open-Window row and its reopen in the same flows — **survives**.
- D1: `lym ide` → `desktophook.AfterWindow` unchanged (`internal/ideengine/spawn.go:238`) — **survives**.
- D2: `OpenAttach` reached from `lym reed window` and `machinecli` `deps.terminal`; both change together; Konsole's stdio is nil today, so nothing holds a pipe — **survives**.
- D2: `konsole --help-all` lists `--separate, --nofork` on this machine — **survives**.
- D3: KWin (same user) reads the script under the 0700 `lymtemp` run dir — **survives**.
- D3: `journalctl --since "@<epoch>.5"` and `"… HH:MM:SS.250"` both parse without error here — contradicts the record's "second granularity" rationale, but the design does not depend on it (noted for the handler).
- D4: `kscreen-doctor -j` here shows an output with `"enabled": false`; D4 filters enabled outputs — **survives**.
- D5/D7: desktop module spec is hub-wide, keys `CategoryMachine` (`internal/desktophook/config.go:39-49`); new machine keys fit — **survives**.
- D7: `LookupWithSource` reads Anchor and Board places only, never the machine layer (`internal/dobbyengine/tasksetting.go:90-124`), so `""` handing over to the machine key holds — **survives**.
- D7: Seed-Child copies `run_settings` into the local layer; a `CategoryNone` key is allowed there — **survives**.
- D7: `checkTaskSetting` falls back to land-notice values for an unlisted key (`internal/dobbyengine/config.go:200-206`) — plan-level wiring, not a finding.
- D8/D12: `lym machine start` `startHub` runs `runBeamStarts` and only then `openWindows` (`internal/machinecli/start.go:253-254`); resumed dobbies' batten reopen runs `lym reed window` before any orch window exists — **breaks** (F2).
- D9: places file is new; only `kwinplace` writes it; no cleanup/teardown/reconcile path touches a hub `.lym` cache; `state.UpdateJSON` serializes writers that go through it (`internal/state/doc.go:1-14`) — **survives**.
- D10: batten Open-Window row/reopen run the command through `verifyrun` with an output writer and WaitDelay (`internal/verifyrun/run.go:122-137`); a detached child started with nil stdio (as `termwindow` starts Konsole) holds no pipe, so the 30 s row bound holds — **survives**.
- D10: when the opening verb runs inside a run, the child is registered under that run's slug, so a teardown's `runproc.End` ends it; bounded, harmless — **survives**.
- D10: the per-user lock — `internal/lock` offers only path-taking acquires plus tracked-file lock paths, and `lymdirs` names only hub/worktree `.lym` homes — **breaks** (F1).
- D11: no existing path rearranges windows — **survives**.
- D12: `lym reed window`'s help text and the desktop template's `window_wait_ms`/`window_command` comments still describe an in-verb wait — **breaks** (F3).
- D12: `batten.open_window_command` defaults to `{lym} reed window …` (`internal/battenshed/template.yaml:2`), which the record states as its bound — **survives**.

**Findings**

F1. MEDIUM, design, `_lym/discussion/decision-record.md` D10 ("one per-user lock through `internal/lock`").
The record names no home for a lock that must serialize every hub's placements. `PATTERN-lym-files` puts lock files under a checkout's `.lym/locks/` with paths only from `lock`, and `lymdirs` declares only per-hub and per-worktree `.lym` directories; a lock under one hub's `.lym` does not serialize across hubs, which the "start every hub" rationale needs. No existing helper gives a per-user, cross-hub lock path (e.g. under the `lymtemp` root or the per-user machine file's directory).
Evidence: `internal/lock/lock.go:62-78` (caller-given path), `internal/lock/tracked.go:19-29` (tracked files only), `pattern/PATTERN-lym-files.md:14-15,94-97`.

F2. MEDIUM, design, `_lym/discussion/decision-record.md` D12 ("run windows come from batten's reopens alongside them and rely on D8's wait for the parent") with D8's 20 s wait.
In `lym machine start`, `startHub` runs `openWindows` only after `runBeamStarts` returns; `beam start --all` brings up each dobby, and each resumed run's reopen runs `lym reed window` from then on, so every run window opens before any orch window of its hub, by as long as the whole beam-start child takes. On a first start (no remembered parent place), a hub whose `beam start --all` runs past 20 s anchors its run windows to the current desktop, not the orch's — the Goal's "on its parent orch's desktop" is missed, and the record does not state this path's length.
Evidence: `internal/machinecli/start.go:253-254` (`runBeamStarts` then `openWindows`), `:306-336`; `internal/battencli/arm.go:884-922`.

F3. LOW, consistency, `_lym/discussion/decision-record.md` D12 / Acceptance criteria `Review:` bullet.
Under D12, when placement applies, `window_command` runs in the detached child with no `window_wait_ms` wait, after the verb returned; `lym reed window`'s help ("runs it next, after the configured wait") and the desktop template's `window_wait_ms` comment ("so the command sees the new window") still state the old behaviour, and the record's Review criterion covers only the new keys' comments.
Evidence: `internal/reedcli/window.go:75-79`; `internal/desktophook/template.yaml:1-2`.

**Tool calls**
1. grep for per-task key lists / `BeamModes` — to see whether per-task readers hard-code their keys (context lacked it).
2. grep for `run_settings` field check / `checkTaskSetting` — to see how a new per-task key's values are judged on load.
3. read `lock.go` bounded acquire + grep `lymdirs` per-user helpers — to find where a per-user lock could live.
4. grep for command producer output capture — to check whether a detached grandchild can hold the Open-Window row's pipe (returned nothing).
5. grep `shedadapters`/`verifyrun` stdout and WaitDelay — follow-up to 4, found `verifyrun` runs the command.
