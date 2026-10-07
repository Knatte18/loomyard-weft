---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D5, settle whether every agent-authored part of the hold notice line is bounded, not only the quoted message start: the line also names each outstanding background task by kind and label, and in `internal/shuttleengine/claudeengine/background.go` a shell's label is the raw `command` (else `description`), which can span lines; `orchengine.QueueNotice` in `internal/orchengine/notice.go` refuses any line containing `\\r` or `\\n`, and D5 says a notifier error is only logged at Warn. Check that D5 and its acceptance criteria state the replacement of control characters, a length cut and a cap on the number of tasks named, for the labels as well as the quote."
  - "In D5, settle whether the quoted message start has a stated fixed delimiter and a position that keeps the agent's words from running into or imitating the way-forward text of the same notice line, and whether an acceptance criterion pins it."
  - "In D4's Bound, settle whether the stated source of the held run's deadline is true for every caller that waits on a held run: callers set `Spec.Timeout` from a `*_timeout_min` config key, a `--timeout` flag, a burler recipe row `timeout_s` or a treadle profile value, with `run_timeout_min` where it is left zero."
  - "Settle whether the set of spawners that receive the hold notice is worded the same in the Auto-mode assumptions and in D5's wiring sentence."
---

Round 3's single gating finding concerned the task-label channel of the hold notice.
The other focus entries are wording checks in the same section.
