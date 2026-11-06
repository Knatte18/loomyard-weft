# A parent cannot withdraw a PATTERN change the decision record declares

`decision-withdraw-pattern` · Task · bug, loom, parsers, pattern · running · Darn-Write

Depends on: [`raddle-motif-finish-clean`](design-raddle-motif-finish-clean.md)

## Seen (orch, 2026-11-05)

The `slack-relay` record, filed by raddle, declared `PATTERN-reap-children` (changes) and `PATTERN-slack-boundary` (adds), while its Delivers line folded both into the live clusters `PATTERN-child-processes` and `PATTERN-agent-addressing`.
The parent recorded the two cluster changes with `lym loom decision add`, but the plan gate (`internal/loomshed/patternplan.go`: `pattern-line-unprescribed`, `pattern-changes-mismatch`) still demanded lines for the old ids, since `decision add` only appends and a `removes` action means removing a PATTERN.md entry.
With the operator's leave the orch deleted the two bullets from the record by hand, committed in the weft and ran `lym loom decision store`.

## Idea

A `lym loom decision add` option that withdraws a declared PATTERN change, striking its bullet in the same commit as the decision that says so; and raddle, when it maps a Sketch id onto a live cluster, writes the part record's `## PATTERN changes` under the cluster ids, so the conflict never reaches Plan.
