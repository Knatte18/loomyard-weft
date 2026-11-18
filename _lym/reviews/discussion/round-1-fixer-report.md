---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The fix key's own drop branch, extended. The false sentence and the criterion at line 104 are deleted. In place of the sentence, D2 now gives where reed's existing rules leave the viewer in each case, so the viewer landing that review turn 2 asked for stays on the record. No new reed select op was designed, because orchcli cannot tell whether the move ended the viewed window. Its strand ops are only Strands, AddStrand, ReplaceStrand and RemoveStrand (internal/orchcli/cli.go:33-47), and the only engine op that selects a window, FollowRun, also turns the follow state on (internal/reedengine/follow.go:30-47).
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md` unless named otherwise.

- **F1 (fixed differently):**
  - In D2, the sentence "The viewer ends on the beam overview window, as after today's replace." is replaced with a paragraph saying that D2 selects no window itself. The paragraph then gives where reed's rules leave the viewer:
    - newest live agent window first;
    - else the beam overview window in case 2;
    - else nowhere in case 1;
    - the runs strand's window added after a move is never selected;
    - the fall-back replace selects that window as today;
    - a viewer elsewhere stays where it is.
  - The criterion "After the move the viewer is on the beam overview window (D2)." is deleted.
  - The proposed new reed select step was not made. The reason is in the frontmatter.
- **F3:**
  - D3 now names the three paths that place a strand in a window of its own:
    - an ungrouped strand;
    - a grouped strand placed apart because its group does not share a window or its shared window is full;
    - a break-out from the shell window.
  - A strand that opens or joins its group's shared window ignores `Title`, and that window keeps its group's name.
  - The D3 criterion now names the grouped placed-apart case and the group-named shared window.
- **F4:** the D4 criterion's condition now reads "while the worktree's run has a seed and a status file that is not done".
- **F5:** D3 now says that a `Title` two windows share only gives two status-bar buttons of one name. It gives the reasons:
  - tmux allows duplicate names;
  - reed addresses an own window by its id;
  - reed finds the shell window by its console pane;
  - `OpenWindow`'s same-name lookup skips `@lym_own` windows.

  It cites the `doc.go` window-order and detached-window paragraphs.
- **F6:**
  - The Constraints bullet drops the `PATTERN-no-hard-coded-choice` label and keeps the rule as the record's own.
  - D3's rationale drops "per `PATTERN-no-hard-coded-choice`".
  - The same label is dropped from the twin rejected alternative in `_lym/discussion/support-log.md`.
- **Self-check follow-up:**
  - The D2 viewer paragraph is scoped per case, as listed under F1.
  - The support log's turn-2 line on the viewer landing no longer says D2 puts the viewer on the beam overview with a criterion.

## Self-check

- fix-consistency, MEDIUM, "never selected" contradicts D2's fall-back replace: fixed. D2 now says the fall-back replace selects the overview window as today.
- fix-consistency, MEDIUM, "nowhere" is wrong for case 2: fixed. D2 now gives the overview fall-back for case 2 and "nowhere" for case 1.
- fix-consistency, LOW, support log line 20 left stale: fixed.
- fix-claims, MEDIUM, "nowhere" is wrong for case 2: merged with the fix-consistency finding of the same kind and fixed.
- fix-claims, MEDIUM, "never selected" is false on the fall-back path: merged with the fix-consistency finding of the same kind and fixed.
- fix-proposal: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None. F2 is a `decision`-class finding held by the parent, and no ruling on it reached this round.
