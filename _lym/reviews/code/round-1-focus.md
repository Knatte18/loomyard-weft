---
round: 1
focus:
  - "internal/loomcli/gotohook_integration_test.go, new row PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn: does it pin the case the decision record's added entry names, a recorded strand on a receiver with no runner, and keep the record whole? Check it against prReviewGotoAdmitted and stopLiveSession in internal/loomcli/prreviewgoto.go."
  - "Does the diff stay inside the added entry's scope, one row in that test and nothing else? The reworked table, the DeepEqual comparison and the Fatalf-to-Errorf change need a check that they widen no existing assertion; the other rows must still assert what they asserted before."
  - "The new row's expected error substring: is the stop failure deterministic with no reed session up, with no dependence on the environment or the machine? Does the test still check the behaviour its name and doc comment state?"
  - "Commit message 350370384: the added entry says the findings are fixed and pinned, so no Root-Cause trailer is owed; check that every claim in the message, such as the hook building its own runner, holds against prreviewgoto.go."
---

Single-commit, test-only diff. The reach and mutation reports show no changed non-test function.
