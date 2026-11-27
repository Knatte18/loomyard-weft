```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and guarantees with concrete counterexamples.

**Findings**

1. **MEDIUM — D3, journal channel.** The record never says how lym ties marker lines to one script run, or how it knows a report is complete.
   - Evidence: placements under the D10 lock run back to back. `journalctl --since @<epoch>` uses second granularity, as the operator's `@$run_at` does.
   - Counterexample: window A's apply ends at t=100.9, and window B's query reads `--since @100`. B then parses A's query and apply lines as its own snapshot.
   - `qdbus6 … run` also returns before KWin has run the script, which is why the operator's script sleeps 0.2 s. A read made straight after it can miss the lines entirely.
   - Fix: each run puts a nonce in its marker lines and ends with an end marker. lym re-reads the journal, within a bound, until that end marker appears, and only then unloads the script.

2. **MEDIUM — D8 and D9, stale process ids.** A remembered pid from before a reboot can match an unrelated live window.
   - Counterexample: after a reboot, `lym machine start` opens a task window before its beam's window. The beam's places entry still holds pid 4321, and pid 4321 is now some other Konsole or app window. D8 takes that window's desktop as the anchor.
   - Fix: the KWin snapshot checks a remembered pid against `resourceClass` konsole plus the window caption or title. Alternatively, the record stores the process start time and checks it before trusting the pid.

3. **MEDIUM — D10 with D11, lock bound.** D10's bounded lock wait is never sized, and D11's wait of up to 10 s for the window may run inside the lock.
   - Counterexample: several runs reach their Open-Window rows together. Each placement holds the lock for Konsole startup plus query, apply and journal reads, which is about 1–3 s and up to 10 s. Later placements hit the lock bound and are skipped with a warning, which is the very case D10 exists for.
   - Fix: wait for the window to appear outside the lock, and size the lock bound above the worst queue. Alternatively, state the skip bound explicitly.

4. **MEDIUM — D7, bound incomplete.** The bound says the override reaches only task and beam worktrees.
   - Evidence: `dobbyengine.LookupWithSource` also reads the prime worktree's own files and the board's default layer, `Dirs{Board}`.
   - Counterexample: `lym config beam --set window_placement=next` on the hub default file sets the mode for every machine. That beats each machine's `desktop.window_placement`, and the operator decided the default is per machine.
   - Fix: state that the prime's files and the hub default layer also override the machine key, or have the lookup ignore the board layer for this key.

5. **LOW — D8 and D6, rank ties and insertion.** "The placed window goes at its remembered rank" never defines what happens when ranks collide or when the rank exceeds the current count.
   - Ranks collide after a cross-hub rearrangement, which D9's bound admits.
   - On reopen after a restart, windows arrive one by one in machine-start order, not in rank order. Inserting by index gives a different order from comparing remembered ranks.
   - Fix: order by remembered rank among the windows present, with a stated tie-break.

6. **LOW — D5, logical size.** "Mode size divided by scale" ignores rotation and the panel.
   - Counterexample: a portrait screen of 2160x3840 at scale 1.5 is computed as 2560x1440.
   - The query script already sees KWin's logical output geometry and `clientArea`, which is the usable area. Using those would be exact.

7. **LOW — D2, `--separate` always on.** Every Konsole window then becomes its own process, which adds memory per window, even when the mode is `off` or the desktop is not Plasma.
   - Fix: pass `--separate` only when placement will run.

8. **LOW — D12, other entry points.** `batten.open_window_command` is operator-configurable. A hub that changes it from `{lym} reed window …` gets no placement for run windows.
   - Separately, `window_command`, such as the operator's arrange script, runs after the apply. It can move windows after D9 has recorded their places, so the record is wrong until the next placement.
   - The record states neither.

9. **LOW — D8, past the last desktop.** "Not placed" leaves the window where KWin opened it, often on the full anchor desktop. That desktop then holds more than its capacity, against D5's "never more than ten" intent.

10. **LOW — D1, environment.** `lym reed window` from the batten Open-Window row runs inside the reed tmux server's environment.
    - A tmux server started without `KDE_SESSION_VERSION`, `XDG_CURRENT_DESKTOP` or `DBUS_SESSION_BUS_ADDRESS`, for example over SSH, silently skips placement.
    - Fix: name this as a warning case.

**Claims attacked**

- D1, never fails a window open: holds. The environment gap is in finding 10.
- D1, capability probe instead of a version parse: holds.
- D2, the pid is unique per window with `--separate`, and `OpenAttach` can return it: holds. `proc.Detach` only sets `Setsid`, so the started process is Konsole itself. The cost is in finding 7.
- D3, results come back through the journal: broken (finding 1).
- D3, one module reaches KWin and children are waited on: holds.
- D4, the priority-1 output is the tiling screen: holds.
- D5, the 2x5 cap and the per-screen capacity numbers: hold for the given screens. Rotation and panel are in finding 6.
- D6, the grid formula never exceeds C columns for any n up to capacity: holds. It is undefined for n above capacity, but that case is unreachable because a full desktop overflows.
- D7, override path and bound: broken (finding 4).
- D7, an unknown value is refused: holds, because `LookupWithSource` refuses one and names the file.
- D8, the anchor lookup: broken by stale pids (finding 2).
- D8, overflow: holds. The past-the-end case is in finding 9.
- D8, remembered rank: underspecified (finding 5).
- D9, per-hub cache, its prune and an absent file reading as empty: hold.
- D9, cross-hub staleness bound: holds as stated. Its interaction with rank ties is in finding 5.
- D10, serialization: holds. The lock bound is broken against D11 (finding 3).
- D11, the bounded poll at four times a second: holds against `PATTERN-daemon-wakeups`, since the wait is bounded.
- D11, a screen change rearranges nothing: holds.
- D12, every path goes through one call: holds for the default config. The configurable command is in finding 8.
- D12, machine start places parents before children: holds as a stated rule.
- Constraint, `desktophook` stays free of window-manager code: holds.

**Tool calls**

- Read `internal/proc/proc_linux.go` `Detach` and `termwindow.OpenAttach`. My context lacked whether the started pid is Konsole's own or an intermediate's.
- Read `internal/dobbyengine/tasksetting.go` and `internal/hubgeom/tasksetting.go`. My context lacked which layers the per-task lookup reads, for D7's bound.
