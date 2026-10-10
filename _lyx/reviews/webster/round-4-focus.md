---
round: 4
focus:
  - "cmd/lyx/testdata/test-budget.yaml and the top-level Test functions in package cmd/lyx: after the merge-in commit 90e02e739 of two branches that each added one top-level test and raised the cmd/lyx row from 72 to 73, does the row cover the package's actual top-level test count, and does TestTestBudget_PackagesStayWithinBudget pass?"
  - "internal/fabricengine/refscanner.go, the IsReadOnlyCommand doc: is each sentence about quoted glob characters true for both git and lyx segments, given that readOnlyLyxSegment rejects a glob character in any lyx word quoted or not and TestIsReadOnlyCommand pins lyx fabric diff '*' as false?"
  - "internal/websterengine/recoverbatch.go, the ErrRecoveryExhausted doc: does it state what the sentinel means without naming a producer, given that RecoverSpawnOrAttach's refusal and begin-batch's existing-report refusal in internal/websterengine/beginbatch.go both wrap it and internal/webstercli/beginbatch.go maps it to recovery_exhausted?"
---

Round 4 checks that the three round 3 doc and budget fixes hold in the committed diff.
