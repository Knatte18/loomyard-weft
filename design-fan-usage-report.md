# Compare fanned and solo review rounds from their usage records

`fan-usage-report` · Note · enhancement, burler

From review-lens-fan PR-Gate (2026-10-08): the Discussion and Plan fans ship off by default until round usage records show what they cost and find.

- A small report verb, or a batten run summary, compares the round usage records (`_lyx/reviews/*/round-N-usage`) of fanned and solo segments, so the operator gets the comparison without reading the records by hand.
- `excludedLenses` drops a whole focus file when the previous round's usage record is missing or unreadable and only logs a Warn, so a lost record silently widens a round back to the full fan; the run's facts should show it.

Operator (2026-10-10): the `standard` fan is turned on for Discussion-Review and Plan-Review in the prime loom config (`discussion_fan`, `plan_fan`) as a trial; after a few fanned runs land, an agent compares their round usage records against the solo rounds before this date.
