---
round: 2
focus:
  - "internal/testkit/tmuxrec test files: does every file name exactly one tier, and does each tier0 file, support files read with it included, stay free of exec.Command and of tmuxkit exports other than Main, as PATTERN-test-speed requires?"
  - "internal/testkit/tmuxrec/session.go (PlaySession) and its callers, among them internal/landingshed/prreview_session_test.go: does the kit assert anything beyond t.Fatalf on its own setup (PATTERN-test-isolation), and does every replaying test still fail when a call goes unanswered?"
  - "internal/testkit/tmuxrec, Replay.answer and Call.read: can a read, a chained argv with a ';' element included, be answered with state recorded before the latest recorded change, and can a reordered, changed or unrecorded call be answered at all?"
  - "internal/testkit/tmuxrec, the recorder and replay of a call's stdin: does a changed text in reed's paste call (load-buffer -b <buffer> -, internal/reedengine/io.go) fail the replay?"
  - "internal/testkit/tmuxrec recordings and session setup: does the tier0 replay depend on the host's PATH, shell location, OS, uid, user or host name, and do the committed recordings match what the current recorder would write?"
  - "internal/testkit/tmuxrec placeholding (formatPIDs, pidPlaceholder, Bindings.placehold, placeholdCall): can a short name or a number equal to a kept pid corrupt a recording, and is each behaviour covered by a tier0 test with independently stated expected values?"
  - "internal/testkit/tmuxrec tests: which tier0 test pins the replay's refusals, its Unanswered report and any report of recorded changes left unused, and do the tests duplicate one another?"
  - "internal/landingshed/prreview_session_test.go: compared with the review base, which asserted the exact registered strand identity, is the registration check as strict, and if not, does a Loosened-Check trailer name it with a measured cause?"
---

The entries cover the kit's tier and assertion rules, the faithfulness of its replay, and the one existing check the change touched.
