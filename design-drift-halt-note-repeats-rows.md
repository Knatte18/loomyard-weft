# Drift-Verify halt note repeats one judge rationale per row and buries the way forward

`drift-halt-note-repeats-rows` · Note · enhancement, drift, loom

Imported from [issue #147](https://github.com/Knatte18/loomyard/issues/147).

## What happened

`lym loom` halted at `Drift-Verify` (state `blocked`, anomaly `escalation-to-human`) because the judge ruled that a change did not match its expectation.
The halt reason, which `lym loom` writes whole into the halt note under the friction directory, is one sentence per changed line of the compared capture.
It listed over a hundred changed lines (a ticked time series), each carrying the same judge rationale, and the note grew to about 44 KB.
The halt output ends with the way forward (`lym drift amend <slug> --entries <file>` then `lym loom resume`, or `lym loom goto --to Darn-Write`) only after all of that text.

## Why it is abnormal

- The same rationale repeats on every row, so the actionable information (the one cause and the two ways forward) is buried.
- A reader of the halt note or the pane has to scroll past the whole list to reach the way forward.
- The same text goes into every consumer of the note (reflection, operator pane), which inflates their context.

## What to do

- Group judge findings that share a case, key and rationale into one entry with a row range and the first and last old/new values.
- Cap the halt reason's length, point to the full findings file by repo-relative path, and put the way forward first.

Provenance: task `dop-pumps-off-negative-spp`; note `loom-halt-4.md`.
