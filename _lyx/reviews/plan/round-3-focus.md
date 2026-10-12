---
round: 3
focus:
  - "Card 8 (`_lyx/plan/08-pause-sequence.md`), the `TestShuttlePairReadings` bullet: does the test reach `shuttlePair.Busy` and `shuttlePair.Background` through the public surface, or does it still call the unexported `readTurn` directly, against `PATTERN-test-economy`? Settle whether its claim that a `Busy` reading `Name == SessionBusy` fails the test is true."
  - "Card 8, the `interruptSession` steps and the `TestPause` rows: with several busy sessions, does the plan make the settle wait once per pass over the covered sessions rather than once per session in series, and does a `TestPause` row with two busy sessions pin the elapsed settle time?"
  - "Card 5 (`_lyx/plan/05-inject-refusal-and-pattern.md`): the two `Inject` tests in `internal/shuttleengine/run_test.go` that reach the told-path refusal, `TestNewRunner_RefusesUnusableToldPaths` and `TestNewDetachedRunner_RefusesUnusableToldPaths`, call `Inject` with a lone `Escape`. Does the card switch them to a non-interrupt input as the decision record's acceptance criterion requires, and is the count in its PATTERN tightening sentence right?"
---
