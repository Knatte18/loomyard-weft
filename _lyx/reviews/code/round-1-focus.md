---
round: 1
focus:
  - "internal/reedengine/overlay.go: does the EAGAIN retry apply to a fork/exec failure only, across `run`, `runStdin` and `output`, never to a tmux that ran and exited nonzero, and does stdin get replayed correctly on a retry?"
  - "internal/shuttleengine/run.go: does the start retry fire only for a `died` attempt whose strand removal succeeded, never for `timeout`, a failed removal, or existing output files, and does each attempt get its own run directory with the last attempt's error returned?"
  - "internal/shuttleengine/config.go and template.yaml: is `start_retries` declared in the template, classified in `ConfigSpec`, and refused when negative, with 0 turning the retry off?"
  - "internal/shuttleengine/send.go: does the strand-removal retry make three attempts with a doubling delay on the run clock, keep the by-hand remedy on final failure, and show the head and the tail of the capture without printing a short capture twice?"
  - "internal/shuttleengine/wait.go and hold.go: is the parent notice withheld only for a held turn end built from a waiting turn end (gated plain-turn-end and ungated shell expiry) on an engine meeting `RequirementBackgroundWake`, checked through `RequirementSupporter`, with a held end that has nothing outstanding still notified?"
  - "Docs and pattern: do `internal/shuttleengine/doc.go` and `internal/reedengine/doc.go` describe the new behavior, and does any PATTERN scan pin (completion-signal sites, shell-guard table) need a matching update?"
  - "Tests: do the new tests use injected sleeps and the fake clock, and does any duplicate an existing test's coverage or leave an added behavior uncovered?"
---

Seed focus from the board entry and decision record against the four commits in the review range.
