```yaml
lens: code-affected-set
outcome: reported
```

Task: list, per change, what the diff's changes force outside the touched lines (lens: code-affected-set).

**Per change**
- New `TestNewFinalize_WiresTheGateFlakyPassesIntoTheLandingTexts` (`finalize_gate_test.go:247`). The set is the `shedfake.MergeShuttle` import (it exists, `shedfake.go:413`), the `newTestDeps`, `newGateFixture` and `writeVerifyLog` helpers, `NewFinalize`, `assembleFinalize` and the `landingTexts` consumers. It opens with `tiergate.Require` and `t.Parallel`. Covered, no finding.
- Rewritten `TestVerifyGate_RerunsATestFailureOnce` (`verifygate_test.go:371`). The set is `fakeVerifyTree.verifyErr`, `writeLog` and `onVerify`, which I read as existing fields. The shared fake and fixtures are unchanged and the other tests' use of them is unaffected. Covered, no finding.
- `rerunErr` handling. `verifyErr` is set on call 2 and never reset. That is harmless because each subtest has its own fixture and `check` stops after the rerun.
- PATTERN and `pattern/`. The diff touches neither (`git diff --stat` is empty for them), and the rubric's declared-changes list is `none`. Nothing to land.
- Docs, stencils, other tests. There is no stencil or doc change, and no other test asserts what the diff changes.
- Commit messages. There is no `Pattern-Departure:` trailer, and none is needed. The two commits and their messages match the diff.
- Test-speed pattern (`pattern/PATTERN-test-speed.md`), checked as the rule document for test changes. Both tests use `tiergate.Require(t, 0)` and `t.Parallel`, so they comply. The redundancy report calls both new or changed tests "not judged" (no coverage map), so there is no test-economy candidate from it.

**Findings:** none.

**Tool calls**
1. Bash: one `grep`/`git diff --stat` call. It confirmed `shedfake.MergeShuttle` exists, checked the test-speed pattern's rules, confirmed the diff has no `PATTERN.md` or `pattern/` change, and listed the `NewFinalize(` test callers. My context lacked all four.
