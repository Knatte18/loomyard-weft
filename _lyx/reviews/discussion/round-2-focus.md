---
round: 2
focus:
  - "Stops by settler in decision-record.md: for each Go driver settle record action (restep, verify-retry, moved, busy-exit, running-exit, hand-over), does the classification say whether that record settles the stop, and does every later settler rule remain reachable? Settle the question against internal/sheddriver/settle.go and drive.go."
  - "The reportengine import constraint versus the way the settler Decision reads go-settle.jsonl: do the Constraints and the Decision name the same set of new imports, given the package doc's rule that each record is read through a tolerant parse in the package that owns its format?"
  - "The Sleep Decision against the acceptance criterion that phase seconds sum to the span: is sleep inside or outside phase and row time, and do the active, wait, sleep and attempt identities still hold when all four overlap?"
  - "The Seconds Decision against internal/statuspane FormatSeconds, which renders a day or more like FormatMinutes: does every markdown duration, including week buckets and long spans, render as the Decision and its acceptance criterion say?"
---

The entries name where to look and which question to settle.
