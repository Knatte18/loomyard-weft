```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, challenge the proposals the fixer carried out as proposed (F2, F3, F4, F5, F6, F8, F9, F11, F13–F17).

**BLOCKING — D9, the hand-move rule from the F6 home/last-desktop split together with F5's pre-wait write**
Changed text: "any other window whose desktop differs from its entry's last desktop was moved by hand, and that desktop becomes its home and its last desktop".
Case: on a `lym machine start` after a restart, window B reopens, and its child writes the pre-wait record (new pid, old last desktop B2), then waits outside the lock. Child A holds the lock, and A's apply report shows B matched by D2, sitting on the desktop KWin opened it on (say B0). B0 differs from the entry's last desktop, so B's home becomes B0. When B's own placement runs, D8's first line sends it to its home, now B0. The window loses its remembered place in exactly the restart flow the Goal names.
Evidence: D9 records the pre-wait pid before any wait; D10 runs other children's apply steps while B waits; the hand-move rule does not exempt unplaced entries.
Needed: exempt any entry that is not placed (its last placement's pid differs from its pid) from hand-move detection.

**BLOCKING — D8, the F2 anchor rule still lets a child read the parent's unplaced live window**
Changed text: "The anchor is read only once the parent's entry is placed … Once the entry is placed, the anchor is the desktop of the parent's live D2 match in the snapshot taken under the lock".
Case: the parent's entry is still placed from the last session, with the old pid, so the child's wait ends at once. Before the child takes the lock, the parent's child writes its pre-wait record (new pid, unplaced). The snapshot under the lock then holds the parent's new window, which matches D2 against the current entry, on the desktop KWin opened it on. The child anchors there and records it for good. That is F2's defect, behind a narrower but plausible race, since machine start opens every window at once.
Evidence: D8 checks "placed" at the end of the wait, but takes the live match under the lock against whatever the entry says by then; D9's pre-wait write happens outside the lock at any time.
Needed: re-check under the lock that the entry is still placed. If it is not, wait again or anchor on the entry's last desktop.

**MEDIUM — D1, the probe-failure outcome from F9's split**
Changed text: "When the probe fails, the window stays where KWin opened it, unplaced and unrecorded".
The probe runs in the child's first query, which comes after D10's "The child records the window (D9)" pre-wait write. So the window is already recorded, as unplaced. Every child of that window then waits the full 180 s.
The D1 acceptance bullet ("unplaced and unrecorded") makes the same claim.
Evidence: D10's step order, and D9 "A placement first records the new window's process id and title, before any wait".

**MEDIUM — D6, the panel clamp proposed by F4, "clamp the bottom row as arrange-to-tiles.sh does"**
On a 1 x n grid the only row is the bottom row, and KWin's tiles fill the whole output, panel included (the script's own comment). So every zone of a one-row desktop runs under a panel at the bottom and gets released: all of that desktop's windows end up untiled, against the Goal's "in a zone of that desktop's KWin custom tiling grid". The D6 acceptance bullet covers only "a 2-row grid", and nothing states the 1-row outcome.
Evidence: `arrange-to-tiles.sh:57-69` clamps any tile whose bottom passes `usableBottom`, whatever row it is in.

**LOW — D8 bound, the 180 s wait from F3**
The bound names only "a parent whose window lym never recorded". A parent that was recorded but whose placement ended in a lock timeout, a window timeout or a failed run also stays unplaced (D9's pre-wait write is never followed by a placing write). Its children then hold placement and `window_command` for the full wait, and that case is unstated.
Evidence: D10 "A child that cannot take the lock in time, or whose window never appears, logs a warning, runs `window_command` (D12) and exits" — it makes no placing write.

No further tool calls made.
