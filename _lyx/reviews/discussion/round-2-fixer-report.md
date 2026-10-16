# Fixer report — discussion round 2

## Changes

- F1 (MEDIUM), fixed as proposed: the Open risks first bullet now states that C13's stacked 3-row half cannot be set up once `reed-add-parentless-placement` lands, naming its cards 2, 4, 5 and 7, and that the hand-added half stays reachable.
  It points to the board note of the new D6, which decides that this task files one note through `lyx board` for rewriting C13's stacked half.
  An acceptance criterion for that note was added.
  Premise checked against `reed-add-parentless-placement/_lyx/plan/` (cards 02, 04, 05, 07).
- F2 (LOW), fixed as proposed: D1's No controlled tmux exceptions bullet now says that pane content judged by S9, C2, C6 and C7 is observed by the operator through `lyx reed attach` and reported back by the agent, as C12's check is.
  D1's Controlled creel exceptions bullet now carries that rule into the section's contents.
- F3 (LOW), fixed as proposed: the Open risks daemon bullet names `daemon.lock`, `daemon.log` and the current journal day file under `_board/.lyx/creel`, matching `internal/hubgeom/creelgeom.go` and `journalWriter.rotate` in `internal/creelengine/journal.go`.

## Disputed

None.

## Deferred

None.
