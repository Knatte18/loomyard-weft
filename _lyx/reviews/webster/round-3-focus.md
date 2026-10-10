---
round: 3
focus:
  - "Test files this branch adds or edits in internal/fabricengine and internal/stencilstore: does any still capture logger output through its own sink (a local buffer type, logger.SetOutput with a hand-written restore, logger.SetVerbosity with a manual restore) instead of internal/testkit/logcapture, as PATTERN-testkit requires?"
  - "The PushRecorded conflict test in internal/fabricengine/boltpullfirst_integration_test.go: does every row that returns the original rebase error assert the board is left clean with no rebase-merge or rebase-apply directory in progress?"
---

Targets the sites the round's fixes touch; the rest of the diff was settled earlier.
