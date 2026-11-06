# Tell a fanned assistant to run its lenses, and record when it did not

The first real fanned assistant seat, Code-Review of `codex-assist-fan-trial`, ran no lenses and nothing flagged it.
Its composed prompt contradicted itself: the seat directive said to work alone and write the report and nothing else, the write surface said the same, and the cluster rules asked for subagents and lens reports.
The assistant followed the top-level restriction, and `round-1-usage.yaml` still read `assist_fan: review-code`.

## The prompt

A fanned assistant now runs its own opening stencil and seat directive:

- `burler-template-assist-reviewer-fanned` names the lens reports on the write surface.
- `seat-directive-assistant-fanned` says "work alone" means no other session, says the subagents are the assistant's own work, and lists every file it writes.
- `burler-assist-cluster-fan` no longer calls the lens reports an exception to the write-surface rules.

A solo assistant keeps `burler-template-assist-reviewer`, `seat-directive-assistant` and their bytes.

`seatengine.Seat` gains `SubagentReports`, which `assistSeat` fills with one concrete path per lens.
The `output_files` marker lists them after the outputs, and a fresh run removes stale ones with the assistant's other files.
`Table.Validate` requires the fanned directive of an assistant that has them, and the solo directive of one that has none.

## The record

An accepted round's `round-<N>-usage.yaml` gains `assist_ending`, the assistant's end marker text, and `assist_lenses_missing`, each lens a done fanned assistant returned no report for.
A lens counts as missing when its file is absent or its report's outcome is `no report`.
A round whose `assist_ending` reads `done` and that lists no lens ran its whole fan.
A missing lens is also logged at Warn, and it never fails the round, since an assistant never gates one.

## Design decisions

- **The lens reports are not `Outputs`.**
  Shuttle waits for every output, and every output of a later seat must be a chair input, so a seat that skipped a lens would never read as done and the chair would be handed files it has no use for.
  The record is what reports the gap.
- **Two stencils, not a conditional.**
  Stencils carry no conditionals and Go carries no prompt text, so the fanned wording lives in its own files, as `burler-template-reviewer-assisted` already does for the chair.
- **`internal/lensreport` now reads one part of a lens report.**
  `Returned` reads the `outcome` key alone, which changes the spec's "No Go code parses" line, and `PATTERN-sole-parsers` lists it as that format's parser.
- **A not-done assistant lists no missing lens.**
  Its `assist_ending` already says what happened, and a stopped or timed-out seat's lens files prove nothing either way.

## Not yet proven

The board entry asks for one real fanned seat in which every lens starts and returns.
This change does not give that proof: a review of this branch runs the binary and stencils from before it lands, so the proof needs a fanned round after landing.
The stated cause is itself a likely one, resting on one run each of a seat that fanned and a seat that did not.
That round's `assist_ending` and `assist_lenses_missing` will show the result.

Lens reports of an assistant stay under the anchor's `.lym`, since only its report is copied beside the review, so an audit of a real fan reads them there.
