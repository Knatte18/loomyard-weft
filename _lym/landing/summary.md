# raddle-propose refuses a note dependency; loop exit notes say why

`raddle-propose` now refuses a `depends_on` that names a note, so the failure surfaces before the operator approves the proposal.
Before, the proposal passed and Raddle-File then refused the entry at filing.

A dead-loop stop whose loop left no envelope now says why its exit is not recorded, and its loop log field shows the loop's own output.
Before, that field held the envelopes of earlier finished steps.

Closes #111

## What changes

- `internal/raddleparser`: a new check, `depends-note`, reports a `depends_on` that names a note on the board.
  Its detail gives the way forward: drop the dependency, or have the operator promote the note with `lym board promote`, then run `lym loom raddle-propose` again.
  `contracts/specs/raddle-proposal-spec.md` lists the check.
- `internal/boardengine`: the refusal text for a dependency on a note named its commands with a bare quote and ended in an empty `with ;`.
  It now spells out `lym board promote '{"slug":…}'` and `lym board set-deps '{"slug":…,"depends_on":[…]}'`, the latter with the remaining dependencies.
- `internal/shedverbs`: the loop log tags each line of a finished child's standard output with `lym shed: step <trace id> output: `.
  The dead-loop stop's excerpt skips those lines, so it shows the loop's own output.
- `internal/shedverbs`: `loop exit: not recorded` now carries a reason.
  Either the log holds no starting mark for the loop, or no process marked the exit, as after an uncatchable kill or a machine stop.
  The earlier landing already records the exit code and signal marks, so this change adds only the reason and the excerpt fix.

## Design decisions

- `depends-note` reuses the board's own rule, so the check and the filing refusal agree on what a note is.
  `raddleparser` holds its own `KindNote` constant rather than importing the board engine.
- A `depends_on` that names a `board_checked` entry gone from the board still passes as landed; only a note present on the board is refused.
- Child output is tagged in the log rather than kept out of it, so the log still holds the full record for an operator reading it by hand.

## To check by hand

Nothing was run by hand.
