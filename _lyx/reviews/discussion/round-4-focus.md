---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D1 and D6, settle what happens when the fixer's run completes as done (its fixer-report exists) before the handoff marker was written while the reviewer is still live: does the round stop the reviewer, remove its strand and return an error, or return with a live half, and does the acceptance criterion on a round never ending with a live half cover that case?"
  - "In D6's resume branch, settle whether `BurlerProducer` has what it needs to stop a live half and to finish an attached reviewer: the live half's strand guid from the non-waiting probe, a strand remover, and one review-ready step (done outcome, fork audit, strict parse, marker write) shared with the engine's reviewer goroutine."
  - "In D4, D7 and Open risks, settle what bounds a fixer write to the review file that the segment judge and the bouncer facts re-read from disk after the fixer ran, given that `Result.Verdict` is parsed once before the marker."
  - "In D5 and the Constraints, settle whether the stated bound on releasing the fixer early and the standalone marker location (under `.lyx` in the told state directory) are stated identically in each place and name the real bound."
---

Questions only; the reviewer judges each against the rubric and reports what it finds.
