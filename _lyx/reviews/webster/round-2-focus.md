---
round: 2
focus:
  - "In internal/burlerengine/handoff.go, `join` ends a round after one half decides it. Settle whether a half whose run ended OutcomeTimeout is stopped through `stopHalf` before `join` returns, whether a failed stop is returned as `ErrHalfNotStopped`, and whether the `join` comment and the internal/burlerengine/doc.go sentence about no half being left live match that behavior."
  - "In `BurlerProducer.Call`, a timeout retry runs `Engine.Run` as attempt 2 with no probe, and `Run` removes the ready marker before starting. Settle whether any path leaves a live attempt-1 reviewer or fixer able to write the marker or edit the target beside attempt 2, given that shuttle keeps a timed-out run's strand and pane."
  - "In the existing failure-rules and shuttle-outcome test tables of internal/burlerengine, settle whether the timeout rows cover the stop of the timed-out half, and whether a new standalone test was added where a row would do (PATTERN-test-economy)."
  - "In contracts/recipes/loom-recipe.yaml, the three `BurlerRound` rows carry a No-model-or-effort-key comment. Settle whether each comment states its fact once and still matches `entries_burler.go`, which rejects model and effort keys as unknown."
  - "In internal/burlercli/cli.go, the burler Long help paragraph describes the two-agent round. Settle whether its line breaks follow the surrounding help text and whether its content matches the engine's behavior."
---

Round 2 reviews the branch's whole range afresh, with the first four entries on the timed-out-half stop and the last two on the wording fixes.
