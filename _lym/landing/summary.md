# Let a fanned assistant seat run its lenses, and record a silent solo run

A review round with an assistant fan (`code_assist_reviewer_fan`) composed a prompt that contradicted itself.
The seat directive and the write-surface line told the assistant to work alone and write its report "and nothing else", while the fan rules asked for subagents and lens reports.
In the first real fanned seat the assistant followed the restriction, ran no lenses, and nothing in the round showed it.

## What changes

- A fanned assistant now runs a fanned seat stencil, `burler-template-assist-reviewer-fanned`, and a fanned directive block, `seat-directive-assistant-fanned`.
  "Work alone" there means no other session, and running the lens subagents is named as part of the seat's own work.
  The write surface and `output_files` list the report and then one lens report per lens.
  A solo assistant keeps its stencil and directive unchanged.
- `burler-assist-cluster-fan` no longer calls the lens reports an exception to "nothing else", since the write-surface rules now name them.
- `internal/burlerengine` picks the fanned stencil when the assistant's fan resolves to lenses, and gives the seat a `SubagentReports` path per lens (new `Seat` field in `internal/seatengine`).
  Those files are cleared before a fresh run like the other assistant files.
- `internal/lensreport` gains `Path` and `Returned`.
  `Returned` reads only the outcome line of a report's opening yaml block, so a missing file, a malformed block and `no report` all count as not returned.
  `twill-lens-report-spec.md` and `PATTERN-sole-parsers.md` are updated to say Go reads that one thing.
- A done fanned assistant that left lens reports unreturned is named on `AssistOutcome.MissingLenses` and logged as a warning.
  The round's usage record (`round-N-usage.yaml`) gains `assist_ending` and `assist_lenses_missing`, reset on each accepted round, so `assist_fan` alone no longer passes for a fan that ran.

## Design decisions

- Lens reports are write-surface files but not outputs, so a missing one does not fail the assistant or the round.
  The record flags it and the caller judges.
- The fanned variant is a separate stencil, not a conditional in the solo one, because the stencils carry no `{{if}}`, and it keeps the solo bytes unchanged.
- A fan the lens library does not define resolves to no lenses, so the seat stays solo and uses the solo stencil.

## To check

- Not run: the proof the task asked for, one real fanned seat in which every lens starts and returns.
  The prompt fix is the likely cause of the solo run, not a proven one, so run a real fanned assistant round and confirm each lens report is saved with `outcome: reported`.
- `assist-record-keys-stale-across-attempts` (design, MEDIUM): `assist_ending` and `assist_lenses_missing` in `internal/shedadapters/burlerreview.go` and `roundusage.go` are reset on each accepted round; check that a later attempt cannot keep an earlier attempt's values.
- `fan-wiring-to-keep-assist-reports-unpinned` (design, MEDIUM): the lens names passed to `keepAssistReports` in `internal/burlerengine/assist.go` come from the fan wiring; check that the fan passed there is the one the seat was built with.
- `stencil-prose-asserted-in-tests` (consistency, BLOCKING): tests in `internal/seatengine` and `internal/burlerengine` assert stencil prose; check that none remain that break on a wording change.
