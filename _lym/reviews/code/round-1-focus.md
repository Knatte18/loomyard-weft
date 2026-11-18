---
round: 1
focus:
  - "internal/shuttleengine/run.go `sendStartPrompt`: does the busy retry loop honor the run deadline, and can a retry type the line twice after a send that typed part of it?"
  - "internal/shuttleengine/run.go `DeliverPendingPrompt` and `typePendingPrompt`: every caller holds `sendMu` as the doc says, and `promptDelivered` is still set on every path it was set on before the split."
  - "internal/shuttleengine/hold.go `retypeLostPrompt`: lock order against `sendMu`, the covered and non-parsing-engine guards, and whether `promptSentAt` is set on every path that types the prompt, including an attach."
  - "internal/shuttleengine/hold.go `retypeLostPrompt` against the board entry's second part: does the diff find and fix the cause of a review strand starting without the turn that names its prompt file, also after a restart?"
  - "internal/shuttleengine/run.go `keepFailureCapture`: the styled and plain capture choice, and that the saved capture on teardown is the failure pane only for a prompt delivery failure."
  - "Scope against the board entry: the entry's first part asks for bounded polling or retry of the idle check; does the diff cover the idle-detection mismatch on the saved `startup-capture.txt` layout, or only the retry?"
  - "Test coverage of the new behavior: the busy retry exhausting its retries, the deadline stop, and the single retype cap; check the two new tests' assertions against each behavior."
---

Seed focus for round 1.
The diff touches only `internal/shuttleengine`: the start-prompt busy retry, the failure-pane capture, and the lost-prompt retype in `Wait`.
