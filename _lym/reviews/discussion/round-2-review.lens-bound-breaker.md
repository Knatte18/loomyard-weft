```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee, ordering promise and existing-code claim in the decision record.

**Claims attacked**
- D1 bound: a tmux server without the KDE variables skips placement, logged once per window. Held.
- D1: with a pre-launch check failing, no KWin call is made and the command line is today's. Held.
- D2: the Linux detach only starts a new session, so the started process is Konsole itself. Held: `proc_linux.go:348-350` sets `Setsid` and nothing else.
- D2: pid, class and caption together name one window. Held.
- D3: back-to-back and concurrent runs are kept apart by the run id, and the script is unloaded by plugin name on every exit. **Broken** when runs share a plugin name (F-A).
- D3: the journal re-read is bounded at 5 s, four reads a second. Held: a bounded wait under a minute is exempt.
- D5: DP-2 gives 2x5 and eDP-1 gives 1x3. Held: 2560/500=5, 1440/600=2, 1536/500=3, 960/600=1.
- D6: past `C` only a 2-row capacity allows; a desktop at or over capacity is never a target. Held.
- D7: the per-task lookup falls back to the hub default and never the machine layer. Held, as the record states it.
- D8: "the 180-second wait covers that gap", with the bound "beam starts outlast the wait". **Broken**: the bound understates the condition (F-D).
- D8 fallback: a child anchoring on the current desktop. **Broken**: the record's own anti-"record it for good" rationale does not hold on this path (F-E).
- D8: the parent wait wakes at most once a second and backs off to 8 s. Held. Other children's writes reset the back-off, but the 1 s floor holds.
- D8: the settled and unsettled anchor rules against the parent's pre-wait write race. Held: a stale settled entry's old pid has no live D2 match, so the anchor falls to its last desktop.
- D9: every write is one `UpdateJSON` that changes only its own fields. Held.
- D9: "An absent or unreadable file reads as empty". **Broken** (F-B).
- D9 bounds on a second `lym machine start`, a moving `window_command`, and overflow. Held.
- D10: the blocking acquire wakes nothing while held, as the Slack relay holder's does. Held: `holder.go:156-193`.
- D10: the per-user lock sits where the machine record and the launcher lock sit. Held: `hubs.go:21`, `desktoplauncher.go:199-207`.
- D10: "the child exits with the acquire still pending, which drops the attempt". **Broken** (F-C).
- D10 bound: "reads no input but its arguments, the places file and KWin". **Broken** (F-F).
- D10: the 120 s lock wait against a burst. Held as far as can be judged: per-placement time is unmeasured, and each script run is bounded at 5 s.
- D12: `lym machine start` opens windows only for worktrees whose beam start succeeded, and runs no `window_command` today. Held: `start.go:343-379`.
- D12: run windows can open before their orch's window. Held: `start.go:253-254`, and hubs start one after another (`start.go:198`).
- D12 bound: a changed `open_window_command` gets no placement. Held.
- Support log: `AcquireWriteLockWithin` retries every 10 ms. Held: `lock.go:17`.

**Findings**

- **F-A** — MEDIUM, design, `_lym/discussion/decision-record.md:59-63` (D3), with `:201-202` (D10).
  - Summary: D3 unloads "by its plugin name" and gives every run a random run id, but never says the plugin name is unique per run. D10 runs the window wait's query scripts outside the lock, so many children load and unload scripts at once during a burst. With a shared plugin name, one child's unload removes another child's loaded script, and the two loads collide.
  - Evidence: the operator's scripts make the name unique per process (`plugin="kwin-tiling-grid-$$"`, `plugin="kwin-arrange-tiles-$$"` in `~/.local/bin/set-tiling-grid.sh` and `arrange-to-tiles.sh`).
  - Fix: state in D3 that the plugin name carries the run id, so concurrent runs never share one.

- **F-B** — MEDIUM, design, `_lym/discussion/decision-record.md:186` (D9).
  - Summary: "An absent or unreadable file reads as empty" does not hold for a write through `state.UpdateJSON`, which D9 requires for every write. `UpdateJSON` aborts with no write when the file does not decode.
  - Counterexample: with a corrupt places file, every pre-wait write fails, so no entry is ever recorded or settled. Every child then waits the full 180 s for its parent, and the file is never repaired.
  - Evidence: `internal/state/state.go:153-154` ("A decode failure on an existing file aborts before mutate runs, and no write happens") and `readJSONUnlocked` at `:127-145`.
  - Fix: say what a write does with an undecodable file (replace it, or move it aside), or state the stall as a bound.

- **F-C** — LOW, design, `_lym/discussion/decision-record.md:205-207` (D10), with `:228-229` (D12).
  - Summary: on a lock timeout the child is said to exit with the acquire pending. D10 and D12 make it first settle its entry and run `window_command`, which `desktophook` bounds at 30 s. If the lock frees in that time, the pending acquire takes it and holds it unused until the child exits, stalling every other placement.
  - Evidence: `internal/desktophook/window.go:35` (`commandTimeout: 30 * time.Second`). The Slack holder releases a late acquire at once instead (`internal/slackengine/holder.go:184-190`).
  - Fix: state that a late acquire is released at once, as the relay holder does.

- **F-D** — LOW, consistency, `_lym/discussion/decision-record.md:156-158` (D8).
  - Summary: the first-start bound names only "beam starts outlast the wait". A run window's 180 s also has to cover its orch window's own wait for the window and its lock queue behind other placements. The orch's child waits on the prime's settlement too, so the chain run, then beam orch, then prime adds to the beam start time.
  - Counterexample: beam starts taking 150 s, plus prime and orch placements queued behind the run windows' own placements, exceed 180 s with beam starts still under the wait.
  - Evidence: `internal/machinecli/start.go:253-254`, and D10's order at decision-record `:200`.
  - Fix: state the bound as the beam starts plus the ancestors' placements.

- **F-E** — LOW, design, `_lym/discussion/decision-record.md:144` and `:158` (D8), with `:174` (D9).
  - Summary: past the wait, the current desktop becomes the anchor. D9 then makes the landing desktop the window's home, because it "had none", so the fallback anchor is recorded for good. D8's rationale (`:155`) claims to prevent exactly that.
  - Evidence: D9 `:174`, "where it landed becomes its home when it had none".
  - Fix: either keep a fallback-anchored window homeless, or state in the bound that the fallback desktop becomes its home.

- **F-F** — MEDIUM, design, `_lym/discussion/decision-record.md:215` (D10 Bound).
  - Summary: the hidden verb's bound says it "reads no input but its arguments, the places file and KWin". By D7, D5, D4 and D12 it also reads the hub's config, runs `kscreen-doctor`, and runs `window_command` as a shell command, for whatever hub path it is handed. The attack-surface bound omits the shell command it runs and the config that names it.
  - Evidence: D12 at decision-record `:228`; `internal/desktophook/window.go:62-76` runs the configured command through the platform shell.
  - Fix: name the config reads and the `window_command` run in the bound, and say what bounds them: the same command `lym reed window` runs today, for any directory it is handed.

**Tool calls**
- Bash: `proc.Detach` on Linux, the board's `run_settings` validation, the per-hub loop in `start.go`, and the `state` read and write helpers. The manifest showed `proc_linux.go` only to line 80, and showed neither the board validation nor the hub loop.
