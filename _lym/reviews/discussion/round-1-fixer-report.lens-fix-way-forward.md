```yaml
lens: fix-way-forward
outcome: reported
```

Task: check whether the ways forward this fix added (steps, criteria, operator actions) can be carried out and actually make progress.

**Findings**

- **MEDIUM:** the new open question about the ring wake's body can't be settled by the step the fix names for it.
  - Changed text: the decision record's Open risks bullet 2, "...and the D3 real-session criterion shows which", and the support log's Q11, "...and the D3 real-session criterion settles it".
  - The criterion the fix added reads "the operator sees the mail at most once, beside one `creel mail` row". It passes whether or not the terminal draws the body, so it can't show which case holds. Q11 would stay open after the smoke run passes.
  - Neither outcome leads anywhere. If the body is not drawn, the operator sees the ring's mail zero times, which falls short of the Goal's "the operator once". Neither the risk nor Q11 says what follows from that result: accept it as a bound, or pick another operator copy.
  - Evidence: Goal, "reaches the model once and the operator once"; the D3 criterion's "at most once"; Open risks bullet 2, "if it does not, the operator sees the ring's mail only through the model's turn".
  - To make the step useful, the criterion should state the expected observation (body drawn exactly once, or not drawn), and the risk should name the decision taken for the not-drawn outcome.

- **LOW:** the new D3 criterion about the daemon stopping has no route to reach the race it names.
  - Changed text: the D3 acceptance criterion "the client leaving or the daemon stopping after the ring was answered but before the outcome was written releases the claim".
  - The window sits between `answerLocked` putting the outcome on the waiter's buffered channel and `awaitOutcome` choosing among `waiter.C`, `watcher.closed` and `d.done` (`internal/creelengine/server.go` `awaitOutcome`).
  - Go's `select` picks at random among ready cases, so no test hits this branch deterministically without a seam that orders it. Neither D3 nor the Notes for the plan writer names one.
  - Under the testing rule that a test passing only most of the time is a bug, this row has no deterministic route as written.

The operator rerun after a Claude Code update can be carried out: the Notes name `TestSmokeCreelHookDelivery`, and it's framed as an operator action. The D1 criterion for a claim over 5 lines is a real-session check through the same smoke test. No finding on either.

I made no further tool calls; everything I needed was already in context.
