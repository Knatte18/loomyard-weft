# Board README: drop the legend text at the top, and show autonomy and PR setting on P0 and every beam

`readme-header-and-mode-lines` · Task · enhancement, board · starting · Seed-Child

## Seen (operator, 2026-11-26)

The top of the board's README holds a long legend (`modeHeader` in `internal/boardengine/beammodes.go`) that nobody needs:
how bold and italic values read, when a mode line is left out, what a beam PR link means, how layers and After work.
The one useful line is "Hub-wide default: autonomy `off` · PR setting `off`."
Autonomy and the PR setting are not shown for P0, the root, or for any beam:
the render states a setting that the root and every beam take from the hub-wide default only in the header, and leaves their mode lines out.

## Wish

- The header keeps the render time, the prime owner and the hub-wide default line, and drops the rest of the legend.
- P0 and every beam always carry their mode line, with autonomy and PR setting, whether a value is their own or inherited, so the operator reads each one where it applies.
