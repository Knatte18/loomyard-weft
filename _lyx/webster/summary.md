# Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

Two batches ran, both recorded `done`.

## Batch 01-seat-table-optional-values

Covers cards 1 to 6, head `9fede5a55`.
The seat table gained its optional-value names and exposed composed prompts.
`MultiLLMProducer` gained a per-call table source and a fresh-spawn preparation.
`loom.yaml` gained `discussion_producer` and `discussion_advisors`.
The chair and advisor stencils shipped, and `loomengine` builds the seat-table producer's table.
The fake seat runner evaluates a table's gate once on a done chair.
The `internal/loomengine` row in the test budget was raised.
No deviations from the declared file-ops were reported.
Informational scope warnings listed touched symbols outside the batch's own target glyphs, mainly in `internal/loomengine`, `internal/seatengine` and `internal/testkit/shedfake`.

## Batch 07-discussion-seats-entry

Covers cards 7 to 9, head `e5c32d832`.
The `DiscussionSeats` registry entry runs the told table on `MultiLLMProducer` behind the commit decorator, and `loomcli` wires the table.
`loomrecipe` builds the Discussion-Write row with the engine `loom.yaml` selects, and `loomcli` wires the choice.
The overview and the recipe spec describe the two producers and the keys that select and shape them.
One deviation was reported: `internal/loomrecipe/seam_enforcement_test.go` changed, because `loomrecipe` now imports `internal/loomshed` and its import allowlist needed the entry.

The plan-level verify gate runs after this turn ends.


## Background shells waited out

Master's turn end was counted while these background shells were still running, which comes after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; they may still be running in the session.

- `lyx gate test --tags integration ./internal/shedrecipe ./internal/loomcli ./internal/testkit/envkit ./internal/battencli ./internal/shedbuild ./internal/loomshed ./internal/loomrecipe ./contracts/specs 2>&1 | grep -v "level=WARN" | tail -25`
