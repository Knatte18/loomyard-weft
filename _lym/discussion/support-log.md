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

- **Match the KWin window by caption.** The tab title is lym's only handle today, but Konsole's caption carries more than the title and changes with the shell; the process id under `--separate` is exact.
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

_No rounds yet._

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
- Q14 (auto-pick): How long are the waits? Window 10 s, parent 20 s, lock 120 s, journal 5 s (D3, D8, D10).
- Q15 (auto-pick): Is a window recorded under `off`? Yes, so a child with its own mode has an anchor (D1, D9).
