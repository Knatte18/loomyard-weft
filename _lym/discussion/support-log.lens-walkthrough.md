```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens report: I ran each decision's mechanism on one concrete case.

**Findings**

- **MEDIUM — D6/D8 (the full test counts the placed window itself).**
  - Case: mode `same`, capacity 3 on desktop B1. B1 holds the orch window and one run window. A new run window opens, and KWin puts it on the current desktop, B1.
  - The query in step 1 lists that new window on B1. The window has one desktop and is not minimized, so D6 counts it: 3 counted windows.
  - D5 then calls B1 full, and D8 passes the window on to B2, though B1 had room for it.
  - The same miscount happens for a reopened window whose remembered desktop is the current one.
  - Fix: the planner leaves the placed window out of the full test on every desktop.
- **MEDIUM — D3 (the journal read has no per-run marker and no end signal).**
  - Case: two placements run back to back under the D10 lock in the same wall-clock second.
  - The second run reads `journalctl --since <start>` at second granularity, as `arrange-to-tiles.sh` uses `@$run_at`. It therefore also parses the first run's marker lines, so the planner gets a stale or mixed snapshot.
  - `/Scripting/Script<id> run` also returns before the script's console.log lines reach the journal. Nothing in D3 says how lym knows the report is complete; the operator's script only sleeps 0.2 s.
  - Fix: every marker line carries a per-run nonce, the script ends with an end marker, and lym polls for that end marker within a bound.
- **MEDIUM — D8/D9 (a parent with no record falls back to the current desktop).**
  - Case: a task window opens with `beam.window_placement` (new) set to `next` through `run_settings`. The prime's mode is `off`, or the prime's placement was skipped as unsupported, lock-timed-out or never-appeared.
  - D9 writes entries only after an apply, so the prime window has no remembered process id and no remembered desktop.
  - D8 falls through to the current desktop, so the task lands on current+1 instead of the orch's desktop + 1 whenever the operator is looking at another desktop.
  - Fix: either record the parent's process id at open even when no apply runs, or state this fallback as a known bound.
- **LOW — D10 (the lock wait has no value).**
  - "A bounded wait" names no value, while each placement holds the lock for up to D11's 10-second wait for the window to appear.
  - Case: two Open-Window rows fire together. The second waits up to about 10 s plus two script round trips. Whether it is skipped depends on a bound the record never sets.
  - Fix: state the lock wait relative to D11's timeout.
- **LOW — D6/D11 (the grid for more windows than capacity is undefined).**
  - Case: after docking onto eDP-1 (1x3), a desktop holds 7 windows. D6's rule "past C, with R at 2" has no meaning when R is 1.
  - D8 never targets a full desktop, so the grid is never built in practice. But D11's claim that "the next placement on a desktop builds that desktop's grid… windows beyond the new capacity are not moved off it" reads as if a rebuild happens.
  - Fix: state that a desktop at or over capacity is never rebuilt.
- **LOW — D8/D9 (a stale process id can name the wrong window).**
  - Case: after a reboot, an old remembered process id is reused by an unrelated KWin client.
  - The parent's window "found by its remembered process id" can then match the wrong window, and the anchor becomes that window's desktop.
  - Fix: also match the window's resourceClass `konsole`.

**Per decision: case used and result**

- **D1:** `KDE_SESSION_VERSION` unset leads to no KWin call and a normal open. Purpose met.
- **D2:** `OpenAttach` runs `exec.Command("konsole", …)` directly, so `cmd.Process.Pid` is Konsole's own process id. With `--separate`, that id is unique per window. Purpose met.
- **D3:** the two-placements-in-one-second case breaks it (finding above).
- **D4:** DP-2 priority 1 and eDP-1 disabled; DP-2 is matched by name. Purpose met.
- **D5:** DP-2 2560x1440 at 500x600 gives 5 columns and 2 rows; eDP-1 1536x960 gives 3 columns and 1 row. Purpose met.
- **D6:** 6 windows on 5 columns give 2x3. That holds, but the count includes the placed window (finding above).
- **D7:** task `run_settings` is `""`, the beam is unset and the prime is unset. The chain ends at the prime anchor, then the board default, then the template `""`, then the machine key. `TaskSettingChain` with an empty slug gives the prime alone, so prime and beam windows resolve too. Purpose met.
- **D8:** `next` from B1 with B2 full lands on B3. That holds, but the miscount and the missing-parent-record cases break it (findings above).
- **D9:** after a restart, windows reopen in the order C, A, B with remembered ranks 2, 0, 1. Insertion by rank against the current order rebuilds A, B, C. Purpose met.
- **D10:** the lock wait value is unspecified (finding above).
- **D11:** a window that never appears times out with a warning. That holds, but the over-capacity wording is misleading (finding above).
- **D12:** `lym reed window` places, then runs `window_command`. `lym machine start` places parents before children, which the D8 anchor needs. Purpose met.
- **D13:** the classification test forces each new key to be classified. Purpose met.

**Tool calls**

- I read `internal/hubgeom/tasksetting.go` and the span of `internal/termwindow/termwindow.go` around `OpenAttach`. My context lacked the chain's prime and empty-slug handling, and whether the started process is Konsole itself.
