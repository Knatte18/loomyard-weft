```yaml
lens: interactions
outcome: reported
```

Task: review the decision record's decisions pairwise through the interactions lens.

**Findings**

1. **MEDIUM — D9 × D6 × D8 (restore order after a restart).** D9 rewrites the rank of every reported window of the hub after each apply, so the original ranks are lost before the remaining siblings come back.
   - Evidence: after a restart, `lym machine start` reopens windows one by one (D12). The first window back, say remembered rank 3, lands on an empty desktop "at the end", and D9 then rewrites its rank to 0.
   - The next window, remembered rank 1, is inserted at index 1, behind the window that was at rank 3, so the order is reversed.
   - The goal "returns to the desktop and zone it last held" is missed whenever more than one window returns to the same desktop.
   - D6 and D9 need a rule that keeps the remembered rank of a window still unplaced, or that orders against the remembered ranks of its neighbours.

2. **MEDIUM — D10 × D11 × D12 (lock bound against wait time).** D10's lock wait is "bounded" with no value, while D11 lets one placement hold the lock for up to 10 s of polling, plus the KWin and journal calls.
   - Evidence: the "start every hub" launcher opens every hub's windows at once. With N windows queued, any window beyond about bound/10 s is "skipped with a warning", so the launcher's own use case loses placements.
   - The record should either keep the window wait outside the lock, or tie the lock bound to the queue.

3. **MEDIUM — D11 × D5 × D6 × D8 (screen shrink).** D11 says the next placement on a desktop rebuilds its grid and leaves windows beyond the new capacity in place. But a desktop over capacity is always full (D5), so D8 never makes it a target: the rebuild D11 promises never happens there.
   - D6 also defines no grid for `n` above the column limit when `R` is 1, which happens with eDP-1 at 1x3 holding 5 remembered windows.
   - The grid for `n` above capacity is undefined, and D11's promise cannot be reached.

4. **LOW — D2 × D9 × D8 (stale process ids).** After a reboot, a remembered process id can belong to an unrelated process. D8 looks up the parent's window "by its remembered process id", so a reused id that is now a non-lym window would anchor the run to the wrong desktop.
   - D9 has no check that the window found is Konsole or that the entry is from this session.
   - Evidence: D8's first fallback assumes a pid that does not match means no open window.

5. **LOW — D1 × D2.** D1 promises that with placement off, or on another desktop, the window "opens exactly as today". D2 adds `--separate` to the Konsole command unconditionally, and the D2 criterion pins it on every command line, so every window gets its own Konsole process even with placement off.
   - The record should state that behaviour change in D1, or add the flag only when placement runs.

6. **LOW — D3 × D6.** D6 orders untiled windows "in stacking order", but D3's query report lists no stacking order, so the planner's input lacks a field D6 needs.

7. **LOW — D9 × D12 (a second `lym machine start`).** A re-run opens a second window for each worktree. D9 then points the entry at the new process id, so the old window counts as a non-lym window (D6) and takes a zone.
   - The record does not mention this.

**Pairs checked**
- D1×D2: broken (finding 5).
- D1×D7: held. An `off` override and a machine-wide `off` both skip placement.
- D1×D9: held. With placement off nothing is recorded.
- D1×D12: held. `window_command` runs after a failed placement.
- D2×D8 and D2×D9: broken on stale ids (finding 4).
- D3×D6: broken (finding 6).
- D3×D10: held.
- D4×D6 and D4×D8: held. The placed window is moved onto the tiling screen, and other screens are not counted.
- D5×D6×D11: broken (finding 3).
- D6×D9 and D8×D9: broken (finding 1).
- D7×D8: held. A beam's setting also places the beam's own window relative to the prime.
- D8×D12: held. Parents are opened first, so the anchor has a fresh process id.
- D9×D10: held. The places write sits under the lock.
- D10×D11×D12: broken (finding 2).
- D9×D12 on a re-run: broken (finding 7).
- D11×D4: held. The placement touches only its target, and moving the new window off KWin's default desktop does not retile that desktop.
- Every acceptance criterion against its decision: held, except D2's criterion (finding 5) and D11's rebuild claim, which no criterion covers (finding 3).
- Constraints against D3, D9 and D10: held.

Tool calls: none.
