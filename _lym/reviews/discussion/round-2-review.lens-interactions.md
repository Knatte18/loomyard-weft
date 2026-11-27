```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. For each pair of decisions in the record that act on the same state, I checked whether both still hold, then checked the acceptance criteria and constraints against those decisions.

**Pairs checked**
- D8 × D9, the anchor read from the parent's entry (settled or unsettled) against the pre-wait write that keeps home and last desktop: **broken**. When the parent's home is full, or the parent's mode is `off`, its home is not where its placement sends it (finding 1).
- D8 × D1, the `off` record-only run as the parent's anchor: **broken**, same defect (finding 1).
- D8 × D7, a parent and a child that resolve different modes: **broken**, same defect (finding 1).
- D10 × D9, the window-wait timeout settles the entry with no snapshot, then a later apply sees the window: **broken** (finding 2).
- D10 lock × D9 `UpdateJSON` lock, nested acquire of two different files: held.
- D10 window wait outside the lock × D3 apply under the lock: held. A KWin script body runs on KWin's one script thread.
- D3 SIGTERM unload × D10 `runproc` registration × D12 "`window_command` on every exit path": **broken** (finding 3).
- D9 pre-wait write × D9 post-apply write interleaving under `UpdateJSON`: held, provided the rules run inside the mutate.
- D9 overflow (home and key kept) × D8 home-full overflow × D6 ordering: held.
- D9 hand-move rule × an overflowed window (last desktop is its landing desktop): held.
- D9 hand-move rule × D12 `window_command` moving windows: held. The next placement reads such moves as hand moves, and D9 states the staleness as a bound.
- D9 renumber × D6 key ordering and current-zone rule, clamped windows included: held.
- D9 renumber of a home desktop × an absent, overflowed entry: held. The absent entry keeps its position.
- D5 full test × D6 count "after placement" × D3 apply setting desktops to the target alone: held.
- D4 tiling output × D3 `sendClientToScreen` × D6 counting only the tiling screen: held.
- D8 "at or over capacity is never a target" × D11 "only the target desktop is rearranged": held.
- D8 past-last-desktop × D9 settling with no apply × D12 `window_command` on a warning exit: held.
- D8 parent wait (180 s) × D12 first-start order: held. The bound is stated.
- D8 settled parent entry from a previous session × D12 run windows opening before the orch: held. The last desktop is the anchor and there is no wait.
- D2 key (Bolt dir and worktree name) × D9 per-hub file holding the worktree name: held.
- D1 pre-launch checks × D12 `lym reed window` and `lym machine start` paths: held.
- Acceptance criteria D1, D5, D6, D8, D10 and D12 against their decisions: held. The one gap is that D12's criterion lists no SIGTERM case (finding 3).

**Findings**

1. MEDIUM, design, `_lym/discussion/decision-record.md:140`
   - Summary: D8 says an unsettled parent entry anchors on its home desktop, "where the parent's own placement sends it". Two cases break that, and in both the child takes the wrong anchor as its home for good (D9:174):
     - The parent's home is full: D8:150 overflows it to the next desktop, and D9:175 keeps its home.
     - The parent resolves mode `off`: D1:34 moves nothing.
   - Example: a prime under `off` with a stale home, and a task under `next`. The child that takes the lock first anchors on the parent's old home, not where the parent sits.
   - Evidence: `decision-record.md:140`, `:150`, `:175`, `:34`, `:174`.

2. MEDIUM, design, `_lym/discussion/decision-record.md:184`
   - Summary: a child whose window never appears in the 10-second wait (D10:201, :207) settles its entry with no snapshot, so the entry keeps the old window's last desktop (D9:170, :185). When the window appears late, the next apply by any child matches it by pid (D2). Its settled entry's last desktop differs from where it sits, so D9:176 treats it as a hand move. Its home becomes wherever KWin opened it, and its remembered place is lost.
   - Evidence: `decision-record.md:170`, `:176`, `:184-185`, `:201`, `:207`.

3. LOW, consistency, `_lym/discussion/decision-record.md:228`
   - Summary: D12:228 says the child runs `window_command` on every exit path, and D9:184 and D10:208 make SIGTERM an exit path that settles the entry. But the child is registered in `runproc` (D10:209), whose `End` sends SIGKILL 5 seconds after SIGTERM. `window_command` may run for 30 seconds. D12:229 and its criterion at :296 list no SIGTERM case, so whether SIGTERM runs `window_command`, and within what bound, is unstated.
   - Evidence: `decision-record.md:184`, `:208-209`, `:228-229`, `:296`; `internal/runproc/end.go` `endGrace = 5 * time.Second`; `internal/desktophook/window.go` `commandTimeout: 30 * time.Second`.

Tool calls: none. The inherited context held everything this lens needed.
