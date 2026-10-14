---
round: 2
focus:
  - "In tools/tokencount/orch_test.go, settle whether `TestCountOrch` pins the orch path's last-line rule: a prime-transcript message id written on two lines, the first with a partial output count and a timestamp outside the tally span, the second with the full count inside it, must be charged the last line's figure and counted once by `readTranscript` in tools/tokencount/orch.go."
  - "In _lyx/landing/summary.md, settle whether the sentence about dropped earlier lines adding to the repeated-lines count is true for each counter it follows: the run counter (`RunTally.Duplicates`) adds them, while `readTranscript` discards the drop count from `lastLinePerMessage`."
---

Both entries target the code and summary as they stand in the committed diff.
