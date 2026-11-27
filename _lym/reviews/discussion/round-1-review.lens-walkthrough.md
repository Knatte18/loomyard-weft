```yaml
lens: walkthrough
outcome: reported
```

**List** (decision, case, purpose)
- D1 — `lym machine start` from the desktop launcher with KDE vars set: met (reed tmux scrub strips only agent identity and trace id, `internal/reedengine/lifecycle.go:461-469`).
- D2 — fresh window titled `T7 [ly:P3] slug`: not met in full — the pid step holds, the caption step's input is unproven (F3).
- D3 — query run read back from the journal: met (live lines read `kwin_wayland_wrapper[2052]: js: <text>`, tool call 1).
- D4 — Neve, DP-2 priority 1: met.
- D5 — DP-2 2560x1440 → 2x5, eDP-1 1536x960 → 1x3: met.
- D6 — keys 2, 0, 1 reopening in that order on B2: order met; the end state misses the panel clamp (F2).
- D7 — task `run_settings` `next`, beam `""`: met.
- D8 — `lym machine start` on a fresh machine, `next`, prime on B0, a beam and its task: not met; the task can anchor on the beam's window before that is placed (F1).
- D9 — reopen after restart with stale old pids, new pid recorded at open: met.
- D10 — window-appear poll at 4/s: not met; the poll needs KWin queries outside the lock (F4).
- D11 — undock DP-2 → eDP-1 with 8 windows on B2: met (B2 full at 3, overflow, B2 never rebuilt).
- D12 — `lym machine start` windows: purpose met, but `window_command` behaviour changes by whether placement applies (F6).
- D3 apply — placed window opened on B0, target B2: not met; no step moves the window to the target desktop (F5).

**Findings**

F1. MEDIUM, design. Location: `_lym/discussion/decision-record.md` D8 parent wait / D10.
Summary: the parent wait ends at the parent's *live* match, which exists as soon as its Konsole maps, before the parent's own placement ran. Case: `next`, a beam's window opens on current desktop B0, target B1; its task's child sees the beam live and waits for the lock; lock order is not parent-first, so the task's child can win, read the beam on B0 and land on B1 instead of B2. "Parents first" (D12) cannot order detached children competing for the lock, so the anchor the step is handed is not the one the decision assumes.
Fix: wait until the parent's entry records a desktop written after its own apply/record-only run, or the parent's child has ended.
Evidence: decision-record.md:117-120, :155-156, :176.

F2. MEDIUM, design. Location: `_lym/discussion/decision-record.md` D3 step 3 / D6 / Notes.
Summary: the record copies `arrange-to-tiles.sh`'s zone fill but drops its panel clamp, which that script needs because a bottom-row window placed through KWin tiles runs under the panel; the script unmanages such windows and moves them with `kdotool`. The journal shows the clamp firing on Neve today (`js: CLAMP:{0a5742a0-…}:4:724:632:640` at 11:15:22). The Notes describe the script only as "filters windows and calls `manage` per zone".
Fix: decide the clamp (or another answer to the panel) in D6 and state the tool it needs; `kdotool` is not among D1's checks.
Evidence: `~/.local/bin/arrange-to-tiles.sh` lines 43-68; decision-record.md:227; tool call 1.

F3. MEDIUM, design (unpinned tool fact). Location: `_lym/discussion/decision-record.md` D2.
Summary: the match requires the KWin caption to hold the lym title, but that title is only passed as Konsole's `-p tabtitle=` and the shell then runs `lym reed attach` (a tmux client); nothing records a real window's caption, and the D3 contract test pins only the pid. If the caption does not hold the title, every child misses its window, times out at 10 s and nothing is ever placed.
Fix: record a real window's caption on Neve, or match on pid and class alone.
Evidence: termwindow.go:96; decision-record.md:42, :47.

F4. MEDIUM, consistency. Location: `_lym/discussion/decision-record.md` D10 / D3.
Summary: D10 has the child wait "for it to appear in KWin, polling at most four times a second" outside the lock, but a KWin sighting is only possible through a D3 query run (load script, read journal until end marker, up to 5 s). The step therefore needs KWin script runs outside the lock that "serializes the KWin steps", and one poll can take far longer than a quarter second. The parent wait has the same gap: it re-reads the places file, but a "live match" needs KWin.
Fix: state that the waits run D3 queries outside the lock and that this is allowed, or use another signal.
Evidence: decision-record.md:54, :119, :155-156.

F5. LOW, design. Location: `_lym/discussion/decision-record.md` D3 step 3 / D4.
Summary: KWin opens a new window on the current desktop; D4 moves a placed window onto the tiling output, but no step sets its desktops to the target. D3's apply only "builds the grid and snaps the windows", and nothing states whether `manage` on another desktop's tile moves the window there.
Fix: state the desktop move in the apply step.
Evidence: decision-record.md:60, :70.

F6. LOW, consistency. Location: `_lym/discussion/decision-record.md` D12.
Summary: `lym machine start` never runs `window_command` today; under D12 the child runs it after placement, and with no placement nothing runs it, so machine-start windows run `window_command` only when placement applies — unstated.
Evidence: `internal/machinecli/start.go` `openWindows` (no AfterWindow call); support-log.md:14; decision-record.md:175.

Tool calls:
1. `journalctl -g "arrange-to-tiles|CLAMP"` plus a listing of `internal/dobbyengine` start files — my context held no live journal line showing the KWin `console.log` shape or a firing panel clamp.
