---
round: 2
focus:
  - "Where a failed gate re-prompt is retried: does `decision-record.md` D6 name a trigger that fires on a tick without new writer events, and does it state how far a send's own windows can overshoot the run deadline?"
  - "Whether the new `stopped` outcome is reconciled with `PATTERN-completion-signal` and attach's file-contract-first check (`dispositionCandidate` in `attach.go`, which only fires for a running record): does D7 say if stop consults `allOutputFilesExist`, and do the Constraints name which side of the pinned-sites tripwire the new sites fall on?"
  - "The D1 signal condition (hold the send while a `turn_start` follows the newest turn end): what bounds a hold when the turn end never arrives, as after an interrupted turn or a crashed session, and does Open risks describe that case correctly?"
  - "The D4 not-started teardown after `ErrSubmissionNotLanded`: what happens if the pane still consumes the prompt before the strand is removed, and does D4 state a bound or a named residual for that window?"
  - "Whether D2's settle wait and D3's submit window are one coherent bound: where each window starts, what error a settle that never converges returns, and whether the Decisions agree."
  - "Whether D7's concurrent-`Wait` rule and the burler acceptance criterion about the recorded `stopped` outcome agree, and which side owns the in-memory run handle update."
---

Entries are written to settle design questions in the decision record, not to cap or pre-state any verdict.
