---
round: 1
focus:
  - "internal/loomcli/amendduringwebster_test.go, the plan-gate assertion: settle whether the gate passes because it reads Webster's state.json and skips the done cards' Creates, or because the fixture never makes the landed Creates collide with the tree (target `sub#FooN` against the `sub/a.go` content). A pass that would also hold with state.json deleted does not pin the issue's expectation."
  - "The same test's state-record assertion: settle whether the decisionVerb call and the shedengine.Goto call touch state.json at all in this fixture, or whether the byte-equality check can only hold trivially because the test's Goto uses a stub producer list that never reaches Webster's code."
  - "The decision record's two Added entries state the test lands as a guard that passes on base: check the test's doc comment and the commit messages claim nothing more (a fix, a root cause, an improvement) than that, and that the commit range carries no unlicensed change outside the one test file."
  - "Test economy: check the new test against existing decisionVerb, Goto and plan-gate tests in internal/loomcli and internal/loomshed for duplicated coverage, and its helpers (newDecisionFake, writeStatusFixture, productJSON, planApprovedReader) for use that matches their contracts."
---

Round 1 reviews one new test file, the whole diff.
The record asks for a test that drives the reported path and lands when it shows no loss of Webster's run record.
