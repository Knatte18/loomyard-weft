# Support log — kde-window-placement

## Interview

Autonomous run: no operator answered, and no advisors were seated.
Every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry, the board list and the touching entry `area-host-exits`; no `Decision for kde-window-placement:` note exists.
Three exploration subagents mapped how lym opens windows, the config and state layers, and the KWin and kscreen facts on Neve.
Findings that shaped the design:

- lym opens Konsole through `termwindow`, keeps only the tab title, and has no KWin or D-Bus code; `desktophook` runs a configured command after `lym reed window` and `lym ide`, never after `lym machine start`.
- The batten Open-Window row and its reopen run `lym reed window`.
- The config stack has a machine layer for `CategoryMachine` keys, and per-task beam settings resolve through `dobbyengine`'s lookup down the beam chain, falling back to the hub default and template, never the machine layer.
  A project entry carries no `run_settings`.
- Neve runs Plasma 6.6.6 on Wayland; DP-2 is priority 1 at scale 1.5, eDP-1 priority 2 at 1.25; ten desktops named B0 to B9.
- The operator's grid and arrange scripts load KWin scripts with `qdbus6` and read results back from the KWin user journal.
- No auto-snap script exists on Neve in any place searched.
- `konsole --help-all` offers `--separate`.

### Turn 2 — design

Settled each "To settle" point and each proposal by auto-pick (ledger Q1 to Q12), then wrote the three files.

### Turn 3 — lens review

Six lens forks reviewed the first draft; every fork returned a report, saved beside this log.
Findings, merged across lenses:

- existing-paths BLOCKING, Open-Window row's 30 s timeout against in-verb placement: fixed; placement moved to a detached child (D10, D12).
- interactions 2, walkthrough LOW, bound-breaker 3, unsized lock wait held across the window wait: fixed; window and parent waits run outside the lock, lock wait 120 s (D10).
- walkthrough MEDIUM, bound-breaker 1, journal lines of back-to-back runs mix and completion is unknown: fixed; run id on every line, end marker, bounded re-read (D3).
- interactions 1, bound-breaker 5, restore order lost when ranks are rewritten: fixed; order keys kept unless the order disagrees, sort by key (D6, D9).
- walkthrough MEDIUM, the placed window counted in the full test: fixed (D5).
- walkthrough MEDIUM, a parent with no record anchors to the current desktop: fixed; every window recorded at open, also under `off` (D1, D9).
- existing-paths MEDIUM, run windows are not ordered after their orch's on `lym machine start`: fixed; bounded wait for the parent's window, ordering claim narrowed (D8, D12).
- interactions 4, walkthrough LOW, bound-breaker 2, stale process id matches another window: fixed; match on pid, resource class and title (D2).
- interactions 5, bound-breaker 7, `--separate` changes windows where placement never runs: fixed; only when D1's checks pass (D1, D2).
- existing-paths MEDIUM, started pid equals KWin pid unpinned: fixed; evidence stated and the contract test pins it (D2).
- interactions 3, walkthrough LOW, grid above capacity undefined and D11's rebuild unreachable: fixed; a desktop at or over capacity is never a target or rebuilt (D6, D8, D11).
- interactions 6, stacking order missing from the query: fixed (D3).
- interactions 7, a second `lym machine start` leaves a duplicate window: fixed; stated as a bound (D9).
- bound-breaker 4, the prime's files and hub default outrank the machine key: fixed; stated as a bound (D7).
- bound-breaker 6, logical size ignores rotation: fixed; KWin's logical output geometry (D5).
- bound-breaker 8, existing-paths LOW, a changed `open_window_command` and a `window_command` that moves windows: fixed; stated as bounds (D9, D12).
- existing-paths LOW, redundant `window_wait_ms` sleep: fixed; skipped when placement ran (D12).
- bound-breaker 9, past the last desktop a desktop exceeds capacity: fixed; stated (D8).
- bound-breaker 10, a tmux server without the KDE variables skips placement: fixed; stated as a bound (D1).
- board-scope 1 and 2, project override and VS Code left out: kept as Open risks for the parent; class `decision`.
- board-scope aside, `PATTERN-config-layers` may need a PATTERN change: rejected; the entry's overview line names no beam keys, so only its background changes, covered by a Review criterion.
- leanness, Auto-mode assumptions restate decisions: fixed; one pointer line.
- leanness, Constraints restate decisions: fixed; restating lines cut.
- leanness, D6 grid-build detail and D3 command lines: fixed; moved to Notes and Constraints.
- leanness, D13 is bookkeeping: fixed; folded into D5 and D7 and the Review criterion.
- leanness, auto-snap search narrative, Notes duplicate, `area-host-exits` risk, Scope duplicate: fixed; trimmed, moved to Constraints, cut.

## Rejected alternatives

- **Match the KWin window by caption.** The tab title is lym's only handle today, and a lym window's caption reads `<lym title> — Konsole` on Neve, but a second window opened on the same worktree carries the same caption; the process id under `--separate` tells them apart.
- **Open the orch windows before the beam starts.** `lym machine start` opens windows only for the worktrees whose beam start succeeded, so the order cannot change; the parent wait covers the gap instead (D8, D12).
- **Poll the per-user lock.** `lock.AcquireWriteLockWithin` retries every 10 ms, which a wait past a minute may not do under `PATTERN-daemon-wakeups`; a blocking acquire wakes only when the lock frees (D10).
- **Clamp the panel row with `kdotool`.** It adds a tool to D1's checks; the apply script sets the frame itself (D6).
- **Record an overflow as the window's place.** A session on a smaller screen would move the window's place for good; the home desktop is kept apart from the last desktop (D9).
- **A session-bus listener for script results.** A KWin script could `callDBus` back to lym, but lym would need its own bus name and a listener; the journal channel works on Neve today and needs neither.
- **A persistent KWin script that watches every window.** It would record moves as they happen, but it needs installing and enabling in `kwinrc` and still needs a return channel; snapshots at each placement catch moves well enough.
- **Places in hub config per machine.** Places are observed state that changes on every placement, not a setting, and a config write per window would churn the machine layer.
- **One per-user places file across all hubs.** It would keep cross-hub ranks fresh, but `PATTERN-lym-files` puts never-tracked caches under a hub's `.lym`; the cross-hub staleness is bounded in D9.
- **Tile only lym's windows.** Other windows would float over the grid; the operator's `arrange-to-tiles.sh` fills zones with every window, so lym does the same.
- **A full top row before a second row.** With 6 windows on 5 columns it leaves one window under five; balanced rows look like the operator's own 2xN grids.
- **Rearranging every desktop on a screen change.** Touching desktops the operator is not working on would surprise more than it helps.
- **Creating desktops when all are full.** The operator keeps ten desktops by choice.
- **A project-level setting.** It needs the board to accept settings on a project entry, which the board refuses by design; raised as an open risk.

## Review rounds

### Round 1

- F1, the 120 s lock wait polls every 10 ms: fixed differently; a blocking acquire that wakes only when the lock frees (D10).
- F2, a child anchors on its parent's unplaced desktop: fixed; under the lock the anchor reads the parent's live window only once its entry is settled, and its home while it is being placed (D8, D9).
- F3, `lym machine start` opens windows after every beam start: fixed; the real order stated, the parent wait raised to 180 s with back-off, the first-start fallback a bound (D8, D12).
- F4, the panel clamp dropped: fixed; the apply script clamps the bottom row through `frameGeometry` (D3, D6).
- F5, concurrent places writes: fixed; one `UpdateJSON` per write, changing only its own fields (D9).
- F6, an overflow overwrites the remembered desktop: fixed; a home desktop kept apart from the last desktop (D8, D9).
- F7, KWin behaviour on non-current desktops unobserved: the apply step's desktop and output move added (D3) and the probe's mechanism replaced; observing each fact on Neve deferred, since it needs the session bus and the operator's live desktop (Constraints).
- F8, D2's pid and caption facts: fixed; observation recorded and `--separate` justified by Konsole's single-process setting (D2).
- F9, the probe cannot decide the command line: fixed; pre-launch checks and the post-launch probe split (D1).
- F10, the waits need KWin queries outside the lock: fixed differently; the window wait runs read-only queries unlocked, a second apart, and the parent wait reads only the places file (D8, D10).
- F11, the per-user lock has no home: fixed; `kwinplace.lock` in `standalonestate.UserStateDir` (D10).
- F12, timeout and kill leave the script loaded: fixed; unload on every exit, SIGTERM unloads, a kill is a bound; a script's body runs whole in KWin, so no half-built desktop (D3, D10).
- F13, `window_command` lost or gained: fixed; run on every exit path of the child, `off` included, `lym machine start` gains it under placement only (D12).
- F14, help and template text: fixed; Review criterion extended.
- F15, a renumber collides with absent keys: fixed; present windows renumbered into their own positions (D9).
- F16, the `--since` granularity claim: fixed; dropped (D3).
- F17, D6's count against `arrange-to-tiles.sh`: fixed; the rationale reworded (D6).

### Round 2

- F1, an unsettled parent entry's home is not where the parent lands: fixed; the anchor reads only a settled parent entry, a child finding it unsettled under the lock releases the lock and waits again within one 180 s bound, and a window placed from the current-desktop fallback stays homeless (D8, D9, D10).
- F2, a renumber rewrites an overflowed window's key: fixed; a renumber covers only the present windows homed on the desktop (D9).
- F3, KWin facts left for later: put to the parent for observation on Neve (creel #2818); the parent declined every probe (#2819), so the facts moved to Open risks with the probes as the implementing run's first step, and the parent's `kscreen-doctor -o` and `kwinrc` readings went into Constraints.
- F4, `window_command` on the SIGTERM exit: fixed; a child ended by SIGTERM runs none and ends one already running (D12).

### Round 3

- F5, D2's resource class `konsole` matches no Wayland window: fixed; the match takes `org.kde.konsole`, read from source and added to the probes, and placement takes a Wayland session only (D1, D2, Constraints).
- F6, a script run can reach another script through a reused id: fixed; every script run holds a per-user script lock until its script has left KWin, under a plugin name unique per run, with -1 a failed load and a leftover unloaded first; collisions with scripts lym does not own are a bound (D3, D10, Constraints).
  The fixer's self-check added: a leftover still loaded fails the next run before its load, an abandoned acquire releases the lock at once, window-wait children share one fresh query report so the lock does not starve them, and D10's total wait counts the script lock (D3, D10).
- F7, the query lacks what D6 orders by: fixed; it reports each window's `internalId`, the current desktop and every desktop's leaf tiles with geometry, padding and windows, and a zone is the leaf that lists the window (D3, D6, Constraints).
- F8, a renumber rewrites an unsettled window's key: fixed; the renumber and its trigger take settled entries only, an unsettled entry counting as absent (D9).

## Question ledger

- Q1 (auto-pick): Which screen does lym tile on? The enabled output with priority 1 from `kscreen-doctor -j`, as the entry proposed (D4).
- Q2 (auto-pick): How is capacity set? A smallest tile size in logical pixels, 500 by 600 by default, capped at 2x5 (D5).
- Q3 (auto-pick): How does a grid grow? 1 x n up to the column limit, then 2 rows of ceil(n/2) (D6).
- Q4 (auto-pick): What happens on a screen change? Nothing by itself; the next placement on a desktop rebuilds that desktop only (D11).
- Q5 (auto-pick): Where do places live? A per-hub cache under `.lym`, not config (D9).
- Q6 (resolved by research): Where is the operator's auto-snap script? Not found on Neve; lym does not depend on it (Constraints).
- Q7 (auto-pick): How does lym talk to KWin? Generated scripts loaded with `qdbus6`, results from the KWin journal, in one new package (D3).
- Q8 (auto-pick): How is a window identified? Konsole `--separate` and the process id (D2).
- Q9 (auto-pick): Where is the default and the override? A machine key with an empty-means-machine per-task beam setting (D7); project override raised as an open risk.
- Q10 (auto-pick): What does `same` do when full? Overflow like `next` (D8).
- Q11 (auto-pick): Are VS Code windows placed? No; raised as an open risk.
- Q12 (auto-pick): Are non-lym windows counted? Yes, as `arrange-to-tiles.sh` does (D6).
- Q13 (auto-pick): Where does placement run? In a detached child, so the Open-Window row's 30 s bound never fails (D10).
- Q14 (auto-pick): How long are the waits? Window 10 s with query runs a second apart, parent 180 s re-reading from one to eight seconds, lock 120 s through a blocking acquire, journal 5 s (D3, D8, D10).
- Q15 (auto-pick): Is a window recorded under `off`? Yes, so a child with its own mode has an anchor (D1, D9).
- Q16 (auto-pick): What does an overflow record? Its landing desktop as its last desktop, keeping its home and order key (D9).
- Q17 (auto-pick): Does lym clamp the bottom row at the panel? Yes, in the apply script (D6).
- Q18 (auto-pick): When does `window_command` run under placement? In the child on every exit path but SIGTERM, mode `off` included; `lym machine start` gains it under placement only (D12).
- Q19 (auto-pick): Where is the per-user lock? In the per-user state directory (D10).
- Q20 (auto-pick): What does a window anchored on the current desktop past the parent wait record? No home, so its next placement anchors on its parent again (D8, D9).
- Q21 (auto-pick): Is an X11 session placed? No; it works as today, since Konsole's resource class differs there and only Wayland runs on Neve (D1, D2).
- Q22 (auto-pick): How does a script run reach its own script? A per-user script lock held from load until the script has left KWin, a plugin name per run, and a leftover unloaded first (D3).
