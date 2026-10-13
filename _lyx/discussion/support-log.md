## Interview

Autonomous run: no operator answers; every pick below is the writer's own and is listed in the Question ledger.

1. Read the board entry `reed-window-order` and the related `stale-binary-daemons` and `logging-cleanup` entries.
   `logging-cleanup` is done (`a90ef61d1`): its watchdog re-stats its binary and retires and respawns on a change, so the watchdog half of the folded-in re-exec work is dropped from scope.
   Its `reapplyLayout` still wraps the whole body in `keepZoomLocked`, so the zoom-flicker fix stays in scope.
2. Explored at module level: reedengine's strands' window (resolved through the Selvage pane), own windows (`new-window -d`, next free index), `OpenWindow`'s hidden windows used by batten in the prime, the anchor vocabulary, the navigation bindings and status bar, loomcli's status and driver strands, orchcli's runs and orchestrator strands, and statuspane's pane loop and `BuildIdentity`.
   `lyx loom start` serves loom and darn, so one status strand covers both; batten's overview is the prime's `runs` strand.
3. Settled the window order (D1–D3): a new `overview` anchor, every other window inserted before the shell, Selvage's window renamed `shell` and kept as the strands' window.
4. Settled focus (D5) on reed state alone, the shell counting as watched.
5. Settled navigation and bar (D6), the checklist frame (D7), re-exec in the shared pane loop (D8), the re-apply fix (D9) and the attach-time migration (D10).
6. Lens review: every fork returned a report. Findings and dispositions:
   - scenarios BLOCKING, consecutive hidden windows defeat a single skip: fixed, hidden windows open after the shell and a hidden landing jumps to the first window or the shell (D2, D6).
   - scenarios BLOCKING, resume relaunches the persisted display, not the new anchor: fixed, D10 states old strands keep their anchor until a fresh-spec placement.
   - premises MEDIUM, `ReplaceStrand` kills the old pane first and keeps the table slot: fixed, replace opens a new window, "newest" is the highest-index live agent window (D1, D2, D5).
   - scenarios MEDIUM, replace across anchors unspecified: fixed in D1 with an acceptance criterion.
   - scenarios MEDIUM, migration only at attach: fixed, also before every window open (D10).
   - scenarios MEDIUM, no resolvable shell window: fixed, plain `new-window` fallback (D2).
   - scenarios MEDIUM, which window-ending paths switch, shared windows: fixed, three named paths, shared window switches nothing (D5).
   - scenarios MEDIUM, new binary dying on the same argv: fixed as an open risk; a pre-exec argv probe rejected below.
   - scenarios MEDIUM, failing applies still toggle zoom: fixed, the last failed box is skipped too (D9).
   - goal-scope and premises MEDIUM, batten overview keeps truncating and reporting: fixed, both overview panes drop report and fit (D7); its row detail stays out with a reason (Scope).
   - goal-scope MEDIUM, orchestrator move under-licensed: fixed, D4 states the reason and the cost is an open risk naming `orch-split`.
   - board-fit MEDIUM, a pane-only helper leaves `stale-binary-daemons` without one: fixed, `internal/selfexec` (D8).
   - goal-scope LOW, watchdog lock gap: fixed, one clause in Scope Out.
   - goal-scope LOW, rounds vs visits: fixed, a review row's visits are its rounds (D7).
   - goal-scope LOW, departures stated: no change needed.
   - board-fit LOW, `orch-split` and Focus: fixed in Open risks.
   - board-fit LOW, `spawn-audit-log` hook: fixed, D8 logs the replacement at Info, which that entry can classify.
   - board-fit LOW, the tackler counts as an agent: accepted, it is an own-window shuttle strand and D5's newest rule covers it.
   - premises LOW, per-visit fields invented: fixed, D7 names the history's fields.
   - premises LOW, wait-line source: fixed, D7 says rewording `reviewWaitingFor` alone does not meet it.
   - premises and pattern-fit LOW, `PATTERN-leaf-packages` does not cover statuspane: fixed, Constraints cite the doc.go import lists.
   - pattern-fit LOW, overview color premise false: fixed, overview strands set no segment (D1).
   - pattern-fit LOW, Windows fallback Warn: fixed, Debug (D2).
   - pattern-fit LOW, failed-stat level: fixed, Debug (D8).
   - scenarios LOW, nothing left on window end: fixed, tmux's choice (D5); exec retry cadence: fixed, once per identity (D8); missing criteria: added.
   - leanness LOW, Constraints restating D5 and D8, no-op Constraint lines, psmux fallback as a Constraint, Goal restating Scope, Auto-mode duplication, harmless sidecar risk, D7 rationale overlap: all fixed by trimming or moving; the sidecar risk moved here: a re-exec'd pane's sidecar still names the old build, so the next `lyx loom start` replaces it once more.

## Rejected alternatives

- **Shell at a fixed high index (`move-window -t :99`).** New windows take the first free index, so a gap left by an ended agent would be refilled out of start order, and 99 is an arbitrary cap.
- **A sort pass after every window change (`swap-window`).** More tmux round trips and transient reorders on screen; inserting before the shell gets the same order for free.
- **`renumber-windows on`.** New windows would still take the next free index, after the shell.
- **The overview as a top-anchored pane in the shell window, larger.** Keeps the cramped shared window the operator asked to leave.
- **An `own-window` strand with an ordering flag instead of a new anchor.** Splits one placement over two fields; the anchor is the closed vocabulary callers already choose from.
- **Retiring the stacked layout, `AnchorTop` and content sizing.** No board entry asks for it, it is a large removal, and an explicit caller can still use it in the shell window.
- **Reed asking the run which row is current to pick the relevant agent.** Makes reed read run semantics it is documented never to read; the newest live agent matches in the common case.
- **A new agent taking focus from the shell.** It would steal keystrokes from an operator typing there.
- **An interactive pager for the overview.** Key handling, alternate screen and mouse handling for a gain copy mode already gives.
- **List at the top, detail below.** A frame taller than the window would show the detail and hide the list and wait line.
- **Re-exec through a respawned pane or a successor process.** A pane's process cannot hand its pane to a child; the pane would die and leave a corpse.
- **Exec without a settle poll.** Could exec a binary the deploy is still writing.
- **A pre-exec probe that the new binary accepts the argv.** Spawns a second process per change for a rare failure `lyx loom start` already recovers.
- **The re-exec inside `statuspane` only.** `stale-binary-daemons` needs the same rule for verbs that are not panes.
- **A repeated skip past hidden windows in the key binding.** tmux bindings cannot loop; placing hidden windows after the shell makes one conditional step enough.
- **Moving the orchestrator into the shell window beside Selvage.** Keeps an agent in the shell window the operator asked to hold the shell alone.
- **Moving an in-flight run's strands at attach.** Moves panes under a live agent; their next relaunch places them correctly.

## Review rounds

_No rounds yet._

## Question ledger

- Q1. Attach and automatic switching: overview or the most relevant agent? Auto-pick: the newest live agent, else the overview (D5); reed reads no run state.
- Q2. Does a new agent take focus from the shell? Auto-pick: no, the shell counts as watched (D5).
- Q3. Fixed shell index or insert-before? Auto-pick: insert before the shell (D2).
- Q4. Does the prime's orch session follow the same order? Auto-pick: yes, the board names batten among the runs (D1, D4).
- Q5. How does the operator reach a row's detail? Auto-pick: detail blocks above the list, scrolled to in copy mode (D7).
- Q6. Where does the re-exec helper live? Auto-pick: a leaf package `internal/selfexec`, called from `statuspane.Pane.Run` and usable by `stale-binary-daemons`; the watchdog's retirement from `logging-cleanup` stays as it is (D8). Revised after the lens review from a pane-only helper.
- Q7. Windows re-exec? Auto-pick: none; restart through `lyx loom start` or `lyx orch start` (D8).
- Q8. What `logging-cleanup` already fixed: the watchdog's binary retirement; not the zoom bracket (Scope, D9).
- Q9. Keep or retire the stacked layout? Auto-pick: keep (Scope).
- Q10. Where do hidden windows go? Auto-pick: after the shell, so the shell is the last visible window (D2).
- Q11. How does reed tell the newest agent? Auto-pick: the highest-index live agent window, since index order is start order (D5).
