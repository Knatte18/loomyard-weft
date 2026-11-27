```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked**

1. D1: a tmux server without the KDE variables skips placement — held (stated bound). The checks never test `DBUS_SESSION_BUS_ADDRESS`, which `qdbus6` needs, but each failure is only a warning.
2. D1: the query script checks `rootTile` takes a desktop — **broken** (finding 4).
3. D2: the started pid is Konsole's and equals KWin's window pid — held, and already today without `--separate`: every live lym Konsole on Neve is its own process whose pid is the KWin window's pid. The record pins it to a future contract test instead of this observation.
4. D2: the caption holds the lym title — held: live captions read `<title> — Konsole`. This contradicts the Rejected alternatives' "changes with the shell" claim (finding 6).
5. D3: the journal's `--since` has second granularity — **broken** (finding 5).
6. D3: the script is unloaded only after its end marker — **broken** (finding 2).
7. D3: every child bounded; script file under `lymtemp` — held.
8. D4: tiling screen = enabled output with lowest priority — held.
9. D5: capacities 2x5 / 1x3 and the full test excluding the placed window — held.
10. D6: counting every window "matches `arrange-to-tiles.sh`" — **broken** (finding 7).
11. D6: grid built and filled as the operator's scripts do — **broken** (finding 3).
12. D7: lookup falls back to the hub default and template, never the machine layer — held (`dobbyengine/tasksetting.go` LookupWithSource builds `Dirs{Anchor}` and `Dirs{Board}` only).
13. D8/D12: run windows "rely on D8's wait for the parent", `lym machine start` opens parents first — **broken** (finding 1).
14. D9: concurrent writes through `internal/state` — held only if every write is an `UpdateJSON` closure, which the record does not say; the early record write's effect on the remembered desktop is undefined (finding 8).
15. D10: the 120 s lock wait suffices for a "start every hub" burst — weak but bounded (a late child warns and exits); not filed.
16. D10: the hidden verb moves only the target desktop's windows and the placed window — held as a stated bound, but the killed-child path breaks it (finding 2).
17. D12: `lym reed window` and `lym machine start` are the only terminal-window paths — held: no other production code runs `reed window` or `termwindow`; the only command-string user is `battenshed/template.yaml:2`.
18. D12: `window_command` still runs after lym places the window — **broken** (finding 9).

**Findings**

1. MEDIUM, design. `decision-record.md` D8 + D10 + D12: the parent anchor is read before the parent's window has been placed.
   D8 takes the parent's desktop from its live D2 match; D10 waits for that match outside the lock, then takes a lock that grants no order. Counterexample: `lym machine start` opens the prime's window, then a beam's window, both new. The beam's child sees the prime's window as soon as KWin maps it (on the current desktop) and wins the lock before the prime's child, which is still in its 10-second window wait. The beam lands at current+1, not after the prime's placed desktop, and D9 records that wrong place, so later reopens keep it. This hits the fresh-machine "start every hub" flow the entry names and batten reopens racing their orch's placement. D12's "parents first" ordering does not help because the children compete for the lock.
   Evidence: `decision-record.md:117-121`, `:154-156`, `:176`; `internal/machinecli/start.go:343-380` opens the windows in a loop and returns.
2. MEDIUM, design. `decision-record.md` D3 + D10: a timed-out or killed run leaves its KWin script loaded and possibly the desktop half-rearranged.
   D3 unloads "only then", after the end marker; a run with no end marker "fails as a timeout" with no unload stated. D10 registers the child in `internal/runproc` when inside a run, so the run's end (`runproc.End`, SIGTERM then SIGKILL after 5 s) can kill the child mid-apply. An apply script that released the tiles (as `set-tiling-grid.sh` does) and is cut off leaves the target desktop's windows floating, and nothing unloads the plugin. The record bounds neither path.
   Evidence: `decision-record.md:54-55`, `:158`; `internal/runproc/end.go:19-24`; `~/.local/bin/set-tiling-grid.sh` release-before-rebuild.
3. MEDIUM, design. `decision-record.md` D6 + Notes: the apply copies only half of `arrange-to-tiles.sh`.
   The operator's script releases every zone whose bottom edge runs under the panel and moves that window to its clamped geometry with `kdotool`, because KWin tiles fill the whole output including the panel. The record and its Notes describe the script only as "filters windows and calls `manage` per zone", so lym's bottom-row windows sit partly under the panel on a 2-row grid.
   Evidence: `~/.local/bin/arrange-to-tiles.sh` CLAMP block; `decision-record.md:93`, `:227`.
4. LOW, design. `decision-record.md` D1: the capability probe is an unpinned tool fact.
   "The query script also checks that `rootTile` takes an output and a desktop" names no mechanism and no observed result. A QJSEngine call to a Qt-invokable method normally ignores surplus arguments, so a one-argument `rootTile` on an older Plasma would answer the output's shared root tile with no error and the probe would pass. The record pins neither the probe nor the version the two-argument form arrived beyond "Plasma 6.4".
   Evidence: `decision-record.md:29`, `:35`, `:185`.
5. LOW, consistency. `decision-record.md` D3 Rationale: "the journal's `--since` has second granularity" is false.
   `journalctl --user -u plasma-kwin_wayland.service --since "@<epoch>.5"` and `--since "<date> HH:MM:SS.250"` both parse on Neve ("-- No entries --", no error). The run id and end marker still hold; the stated reason for them is wrong.
   Evidence: manifest's journalctl check; `decision-record.md:65`.
6. LOW, consistency. `support-log.md` Rejected alternatives + `decision-record.md` D2 Rationale: "the caption … changes with the shell" is false on Neve; every live lym window's caption is `<lym title> — Konsole`. And without `--separate`, each lym window already runs in its own Konsole process with the KWin pid, so the stated reason for `--separate` ("so every lym window has its own process") is not established either.
   Evidence: `kdotool`/`pgrep -a konsole` output (tool call 1); `support-log.md:61`; `decision-record.md:41`, `:45`.
7. LOW, consistency. `decision-record.md` D6 Rationale: "Counting every window matches `arrange-to-tiles.sh`" is false — the script counts a window whose `desktops` list *includes* the desktop (so a window on all desktops counts); D6 counts only windows whose desktops are *exactly* that one.
   Evidence: `~/.local/bin/arrange-to-tiles.sh` `onDesktop` loop; `decision-record.md:90`, `:97`.
8. LOW, design. `decision-record.md` D9: the early record write's effect on an existing entry is undefined.
   "A placement first records the new window's process id and title, before any wait" does not say whether that entry keeps the worktree's remembered desktop and order key. If not, a placement that then fails (lock timeout under the start-every-hub burst, window never appears) loses the remembered place, against D9's "a window not yet reopened keeps its place".
   Evidence: `decision-record.md:136-141`, `:157`.
9. LOW, design. `decision-record.md` D12: `window_command` is lost whenever placement fails after the checks pass.
   Once D1's checks pass, the opening verb leaves `window_command` to the child; D10's child "logs a warning and exits" on a lock or window-wait timeout, and D1 also stops a run as unsupported — in each case the operator's command never runs, where today it always does. The same change starts running `window_command` for `lym machine start`'s windows on Plasma 6 only, where today machine start never runs it (`support-log.md:14`).
   Evidence: `decision-record.md:157`, `:175`, `:179`; `internal/reedcli/window.go:129-135`.

**Tool calls**
1. One call: grep for other production callers of `reed window` (manifest lacked it for claim 17) plus `kdotool`/`pgrep -a konsole` for live KWin pids and captions (manifest lacked them for D2's pid/caption claims).
