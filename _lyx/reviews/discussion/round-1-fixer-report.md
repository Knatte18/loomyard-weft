# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): D3 now sends its own restored-prompt clear sequence, declared in `claudeengine` beside an unchanged `ClearInputSequence`.
  The sequence is one `C-u` per line of the post-settle read with a `BSpace` between each pair, and D3's Bound states it holds no Escape, `C-c` or Enter, only deletes box text, and is at most two keys per line.
  Acceptance criteria gain the multi-line restored prompt emptied on the shuttle fake, and the second Open risks bullet now names the remaining risk: a release moving the caret or changing how `C-u` and `BSpace` edit a multi-line box.
- F2 (fixed as proposed): Auto-mode assumptions gain a D3 bullet that Claude draws the restored prompt, caret at its end, within `InterruptSettle`, confirmed from a live capture.
  D3 now keeps the worker suspended for one more settle and reads again when the post-settle read is empty, states that a later prompt meets the worker's pre-write guard as an empty box or a draft, and bounds a paste landing before a late restore by those two settles, with the stopped turn's own prompt re-sent as the worst outcome.
- F3 (fixed as proposed): D1 states that the caret, side bars and a lone `%` are stripped before the style check, so placeholder-styled text beside a lone `%` reads empty; the Bound line says every non-blank cell of the box's text once those are stripped.
- F4 (fixed as proposed): D5 selects the start header by the spawned process's pid; the Bound's last clause says a concurrent client's truncation can erase this start's lines and the give-up error then says the file holds no line from that start; the criterion reads never a line from another start.
- F5 (fixed as proposed): D2 states the progress line is a wire message the client keeps and writes to stderr, and the verb prints one stdout envelope carrying the last reason and its time when it ends expired or timed out.
- F6 (fixed as proposed): the D2 criterion adds the `expired` record's stall reason and the too-short reason naming an allowed `lyx` command; the D8 criterion adds help text stating the privilege rule as `doc.go` does and the resolved sender on every verb that prints one.

## Disputed

None.

## Deferred

None.
