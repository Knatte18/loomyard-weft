---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/raddleparser/check_test.go:186
    summary: The contained refusal-detail test lacks its required keep marker.
    fix: Add //testtiming:keep immediately above TestCheck_DetailNamesTheWayForward, naming its refusal-reason and way-forward text assertions that TestCheck does not make.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/raddlecontext_integration_test.go:110
    summary: The new context wiring has no assertion that configured junction names are refused.
    fix: Extend TestRaddleBoardContext by configuring a nonstructural junction name in the forged hub fabric pathspec and asserting that got.ValidateSlug refuses that name.
    origin: lens:code-tests
---

### [BLOCKING] Record why the contained detail test remains

**Location:** internal/raddleparser/check_test.go:186

**Issue:** The current redundancy report lists this test as contained by TestCheck.
Its detail assertions preserve behavior that TestCheck's check-name comparisons do not cover.
PATTERN-test-speed requires a keep marker for a retained contained test, and the rubric requires this finding.

### [MEDIUM] Pin configured junction reservations through the context

**Location:** internal/loomcli/raddlecontext_integration_test.go:110

**Issue:** The added table checks length, root reservation and separators, but no configured junction name.
Replacing fabricConfig.Dirs() with nil in raddlecontext.go:89 leaves these assertions passing.
TestValidateWorktreeSlug checks the validator's explicit junction argument, not its connection to hub configuration.

## Review evidence

The extraction preserves NewTask's validation order and error text.
The hook reaches project and part slugs through the existing proposal and approval routes.
The existing parser assertions remain intact.
TestNewTask independently pins the literal 32/33 boundary, so the context test's use of MaxSlugLength is not tautological.
The Root-Cause trailer names the missing validation the diff supplies.
No correctness, affected-set or scope findings were returned by those lenses.
