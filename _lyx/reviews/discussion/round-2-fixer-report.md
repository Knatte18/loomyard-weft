# Fixer report — Discussion-Review round 2

All five findings fixed; `lyx loom validate-discussion` exits ok on the edited pair.
Round 1's seven fixes were re-checked and none regressed.

## Changed

- **F1 (BLOCKING), bookkeeping on disk.**
  D3's delivery record now holds the gate prompt count, the last-prompt time and the waiting-notify count, and its rationale says the throttle and cap continue across a restart and `notify` reaches the gate through the file.
  D4's pending row updates those counts when it carries a prompt.
  D5 says the closure increments the count and stamps the time, and that `notify` adds a waiting notify which the closure carries and decrements outside the cap and throttle.
  D5's bound now says it holds across driver restarts.
  D6's `notify` writes the waiting notify, and "per run" became "per invocation".
  A new acceptance criterion covers a restart after two undelivered prompts and a `notify` run while the driver is down.
- **F2 (MEDIUM), failed send and the cap.**
  D5 decides that a prompt counts when the closure hands it to the loop, so a failed send spends one of the three, and its Warn names `notify`.
  The failed-send acceptance criterion says it counts toward the cap.
- **F3 (MEDIUM), finalize side effects.**
  D2 says `finalize` opens no request and consumes no verdict.
  D4 says a finalize-time call carries no prompt, marks an open request expired, reports `waiting`, and logs a Warn naming an unconsumed reject's review file.
  The deadline acceptance criterion covers it.
- **F4 (LOW), superseded round.**
  D3 marks an earlier round without a verdict superseded when a new round opens; verbs, closure and status act on the latest round.
  D9 reports only the latest round, and a new acceptance criterion covers it.
- **F5 (LOW), notify and delivered on a closed request.**
  `notify` now refuses on a verdict-recorded or expired request, and `delivered` refuses on an expired one.
  The refusal acceptance criterion names which verbs each refusal applies to.

The support log gains two rejected alternatives (counting only successful sends; keeping the counts in memory) and Question ledger Q13.

## Deferred

None.
