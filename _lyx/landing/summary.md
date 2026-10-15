# Run status pane: seconds, round totals, gate time, entry numbers

The per-run status pane (`lyx loom checklist`, drawn for loom and darn runs) now says where a run's time went.
Spawned review sessions now carry the segment they serve in their names.

## What changes

- Closed times draw to the second: `45s`, `12m05s`, `1h05m12s`, and from a day on as before (`2d03h`).
  A time that includes the open visit stays in whole minutes, so a frame whose only change is the next second is byte-identical and the pane does not redraw (it wipes the scrollback on each redraw).
- Each `round <n>` line of a review block shows the sum of its visit lines' times.
- Entries are numbered in recipe order (`[x] 3. Discussion-Write 4m12s`).
  A review segment is one entry and takes one number.
  A row drawn only from history, and the `recipe unreadable` fallback, carry none.
  Rows come from one lookup over every embedded recipe, batten's included.
- Batten's prime runs overview prefixes the stage cell with the number: `7. Land`, and `6.5 Plan-Write` for the child's row while batten is in Run-Shed.
- Under a visit with gate spans, one line per gate entry reads `gate <name> <time>`, then ` refused <n> rework <time>` when it re-prompted.
  A visit with only the legacy `gate_attempts` keeps the old suffix.
- A gate row's refusal (PR-Gate, for one) marks the visit that follows as ` rework for <gate row>`, and the gate row's line adds ` rework <time>`.
  Bounces inside a segment with a Bouncer row are not attributed this way; the round totals show that rework.
- The Bouncer (seed, judge) and Burler (review, fix) rows take an optional `role_prefix`.
  The loom recipe sets `discussion`, `plan`, `webster` and `merge`, the darn recipe `darn`, so strands, windows and bar buttons read like `ly:<slug>:plan-burler-review`.
  A recipe test pins that every such row sets the key, that a segment's rows agree, and that no two segments share a value.
  A standalone `lyx burler run` keeps its unprefixed roles.

## Design decisions

- Shuttle's wait loop records per-entry gate spans (kinds gate, held, rework) with an injected clock, beside the attempt count.
  Held time replaces the re-evaluations run while an entry is pending, so no two spans of one entry overlap and nothing is counted twice.
  A span still open at finalize ends there and counts.
- Spans are stored on the shed history entry as an optional list beside `gate_attempts` (tolerant record; old entries read as before).
  The Burler review row and the Webster row now copy their gate count and spans too, and PR-Rework's exits keep theirs.
  Folded entries append the spans of the entries folded into them.
- The spans live in `Wait`'s memory, so a gate re-attached after a driver restart reports only the spans from the re-attach on.
- Entries written before this change have no spans and draw as before; a session live across the binary change keeps its unprefixed name until it ends.
- Token tallies grouped by role split per segment, so they no longer line up with runs from before the change.
- `internal/shedrecipe`'s import allowlist gains `internal/agentname`, which validates the prefix; the `internal/loomrecipe` test budget rises from 54 to 55.
- The runs overview's and the orch's time columns keep whole minutes.

## To check by hand

- A live run's pane: seconds on closed rows, round totals, gate and rework lines and the numbers, in a loom and a darn run.
- The batten overview's dotted numbers while a child run is in Run-Shed.
- Two review sessions of different segments in reed's bar, to see the prefixed names.
- Review findings left open:
  - The probe assertion in the `shedadapters` attach test (`attachprobe_test.go`) may be unreached, so it may not pin what it claims.
  - The order in which `shuttleengine` finalizes open spans (`gate.go`) when a run ends with several open: confirm the spans come out consistently.
