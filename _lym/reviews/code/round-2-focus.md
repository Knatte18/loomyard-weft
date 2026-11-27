---
round: 2
focus:
  - "internal/landingshed/fixerrun_test.go and the ancestry query in internal/landingshed/fixerrun.go (fixerRun.run): would the tests fail if the two arguments of the IsAncestor call were swapped, and is the error return of a failing query reached and checked?"
  - "internal/landingshed/fixerrun.go (sessionRef) and its test fixture fakeFixerSeats: are the answers for a result with no session id and for an empty engine each reached by a test that asserts the halt text?"
  - "internal/landingshed/finalize.go, the fix review halt for a changed task branch, and its row in TestFinalize_GateAndFixer (internal/landingshed/finalize_gate_test.go): does the row assert the session reference the halt names?"
  - "internal/gitrepo/pushbranch_integration_test.go and recordLandedPush in internal/gitrepo/push.go: is the trace line asserted for both remote-branch delete paths (DeleteRemoteBranch and DeleteRemoteBranchLeased), and does the test file's header and doc comment match what the tests check?"
---
