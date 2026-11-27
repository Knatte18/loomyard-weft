```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced every existing code path that writes, resets or reads the state the record's decisions introduce. The two that break are the batten Open-Window row's 30 s timeout and the window order on `lym machine start`.

**Findings**

- **BLOCKING — D1, D10, D11, D12: placement can push the batten Open-Window row past its 30 s timeout, and the record never mentions that limit.**
  - Evidence: `internal/battencli/arm.go` `reopenWindow` runs the Open-Window command with `openWindowTimeout = 30 * time.Second`, "the default of any Command row". The row runs `lym reed window`, and D12 now places the window inside that verb.
  - Inside that verb, the time adds up:
    - the lock wait from D10 (bounded, but no bound is given);
    - up to 10 s waiting for the window to appear (D11);
    - the load, run and unload calls through `qdbus6`, plus the journal reads;
    - then the existing `window_wait_ms` of 1.5 s;
    - then `window_command`, which may take up to 30 s.
  - On resume after `lym machine start`, many runs reopen at once and queue on the D10 lock, so later runs exceed 30 s. A Command row that runs over its timeout fails, so D1's promise that placement never fails a window open does not hold here.
  - Fix: give the record a total placement budget that fits inside the row timeout, with the lock wait included. Alternatively, have `lym reed window` hand placement to a detached child and return as soon as the launcher has started.
- **MEDIUM — D12: "`lym machine start` places a parent's window before its children's" does not hold for run windows.**
  - Evidence: `internal/machinecli/start.go` `openWindows` opens windows only for worktrees `planStart` started, which are the prime and the beams. Task windows come from batten's `reopenWindow`, which runs when `lym beam start --all` resumes the runs. Those calls run alongside `openWindows` and are not ordered after it.
  - D8's fallback, the parent's remembered desktop, saves the case where a place is remembered. On a fresh machine, a run placed before its orch window exists falls to the current desktop and not the orch's desktop.
  - Fix: either state that the ordering covers only the prime and beams and that run windows rely on D8's fallback, or wait for the parent's window to appear (bounded) before placing a run.
- **MEDIUM — D2: the claim that the started process id equals KWin's window `pid` is not pinned.**
  - Evidence: `proc.Detach` only sets `Setsid`, so the pid lym gets back is Konsole's own. The record pins only `--separate` from the help text. It records no observed match between the started pid and the window's `pid` under Wayland, and no check that `konsole --separate` stays in the foreground instead of handing off and exiting.
  - The rubric's "unpinned tool fact" rule applies. Fix: record one observed run on Neve, or make the tier-0 recording carry it.
- **LOW — D12 with D6: `window_command` runs after placement and can undo lym's grid.**
  - Evidence: an operator's arrange script set as `window_command`, such as `arrange-to-tiles.sh`, refills zones in its own order on the active desktop, not the target desktop. The record says only "unchanged".
  - Fix: state that `window_command` runs after placement and may rearrange the windows.
- **LOW — D11 with `desktophook`: the `window_wait_ms` sleep (1.5 s) still runs after placement has already waited for the window.**
  - Evidence: `desktophook.AfterWindow` sleeps before running the command, and the record keeps that order.
  - It is redundant time on top of the BLOCKING finding's total. Fix: skip the sleep once placement has seen the window, or say why it stays.

**Paths traced, per decision**

- **D1**
  - The Windows and WSL launchers in `termwindow` never reach placement: survives.
  - The batten Open-Window row and its reopen under the 30 s timeout: breaks (BLOCKING).
- **D2**
  - `konsoleArguments` gaining `--separate`: survives. It is only reached through `OpenAttach`.
  - `proc.Detach` returning Konsole's own pid: survives.
  - The `openTerminal` seams in `reedcli/cli.go` and `machinecli/start.go`, and the wiring in `machinecli/cli.go`, must carry the pid: a signature change only, survives.
  - The attach line closing the window on exit 0, then a reopen through `lym reed window`: survives, because the remembered place is reused.
- **D3**
  - `desktophook.AfterWindow` callers (`reedcli/window.go`, `ideengine/spawn.go`): survive, since `desktophook` stays free of window-manager code.
- **D4 to D6:** no existing lym path reads KWin tiling state, so nothing competes. The only conflict is an operator `window_command` rearranging afterwards (LOW).
- **D7**
  - A task's `run_settings` copied into the local layer by Seed-Child: survives, the same path `run_watch` takes.
  - `ValidateMachine` refusing keys that are not machine keys: survives.
  - `TestModules_EveryTemplateKeyClassified`: survives with D13.
  - The machine layer needs `Dirs` with both `Board` and `Anchor`. `reedcli` uses `fabricengine.ConfigDirs`, which carries both. Survives, assuming `machinecli` gets the same `Dirs`.
- **D8**
  - Run windows reopened by batten alongside machine start's orch windows: the fallback holds when a place is remembered and misses on a fresh machine (MEDIUM).
- **D9**
  - `cleanupcli`'s sweeps (logs, the `lymtemp` run directories, `.scratch` directories): none removes `BoltDir(hub)/.lym`, so survives.
  - `weaverengine/reset.go` matched a `RemoveAll` but is weaver-scoped. Not read further.
- **D10**
  - `runStart` in `machinecli` handles hubs one after another, so it does not contend for the lock with itself.
  - Concurrent batten reopens contend for it and compound the timeout (BLOCKING).
- **D11:** `desktophook`'s sleep after placement is redundant (LOW).
- **D12**
  - `battencli` `reopenWindow`, the Open-Window row and `lym reed window`: covered, but under the timeout (BLOCKING).
  - `machinecli` `openWindows`: covered. The ordering claim is wrong for tasks (MEDIUM).
  - `ideengine` VS Code windows: out of scope as stated.
- **D13:** additive template keys under reconcile: survives.

**Tool calls**

1. Grepped the `OpenAttach` and `openTerminal` callers and read the launcher and attach-line code in `termwindow.go`, because my context held only a summary of the launch path.
2. Grepped `internal/proc` for `Detach` and the scope code, and read `machinecli/start.go`'s window loop, to check whether the started pid is Konsole's and how windows are ordered.
3. Read `proc.Detach` and tried to read batten's `reopenWindow`, because the pid question and the reopen path were not in my context.
4. Grepped batten for the Open-Window timeout and the `desktophook.AfterWindow` callers, because the row's time budget was not in my context.
5. Grepped for sweeps that remove `.lym` or `BoltDir` content, to check whether the D9 cache survives cleanup.
