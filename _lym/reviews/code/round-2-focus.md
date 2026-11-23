---
round: 2
focus:
  - "tools/godocreflow/split.go `layoutParagraph`: can any comment line the layout changes still reach the limit (`commentlint.TextReachesLimit`) when the written paragraph had no such changed line? Check both a partial split of a legacy over-limit line and the final `joinRejectedBreaks` rejoining a cut that `splitLongLine` accepted."
  - "tools/godocreflow/reflow.go `planEdit` and tools/godocreflow/main.go `reportLongLines`: does the run name every prose-paragraph line that still reaches the limit, including in a group needing no edit and in a file with no edits, and never name a list-item, heading or indented-code line? Are the printed line numbers those of the reflowed file?"
  - "tools/godocreflow/reflow_test.go: does a test pin the reported line numbers, and does a row tell `splitLongLine`'s latest-fitting-break choice from taking the first accepted break?"
  - "_lym/landing/summary.md `## Evidence`: does every improvement claim carry before and after figures measured on the same tree, with the line-length count current for the final code?"
---

Focus targets the code that round 1 changed; the settled remainder of the diff needs no re-review.
