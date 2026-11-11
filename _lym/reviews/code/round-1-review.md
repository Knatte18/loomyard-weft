---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/adoptcli/check.go:333
    origin: lens:code-tests
    summary: the new failed-without-FailedTier branch of checkMarkOf has no test row
    fix: In internal/adoptcli/check_test.go add a row to TestRunCheckMarks, or a checkMarkOf case, with a failed record that has ended tiers and a nil FailedTier, and assert the mark is the failed state with the text "adopt check of <subject> failed; lym beam status shows why".
  - id: F2
    severity: LOW
    class: design
    location: internal/reedcli/cli_test.go:83
    origin: lens:code-tests
    summary: no test pins that attach and up still pass the guard in a legitimate worktree, or that down still runs in a refused checkout
    fix: In TestRunCLI_AttachAndUp_RefuseFabricCheckouts, or a sibling row set in internal/reedcli/cli_test.go, add a row that runs attach or up in an ordinary task worktree built as a bare .git directory and asserts the error does not contain the refusal text. Add a row that runs down in the Bolt checkout and asserts the same.
---

### [LOW] New failed-without-FailedTier mark branch is untested

**Location:** internal/adoptcli/check.go:333
**Issue:** `checkMarkOf` gained a branch for a failed record that has tiers but no `FailedTier`, and no test row reaches it. The rubric's test-economy bullet counts a behavior the diff adds with no coverage as a finding.

### [LOW] Accepted path and unguarded verbs of the reed refusal are untested

**Location:** internal/reedcli/cli_test.go:83
**Issue:** The refusal test has refusal rows only. A guard that over-refused a prime or task worktree would still pass it, and nothing shows `down` still runs in a refused checkout, which the doc and the Goal rely on. The focus directive asks that a legitimate worktree stay accepted.

## Rejected

- Split of the code-tests finding about `down` into its own finding: merged into F2, same test and same location.
- Concern that "adopt-check tab" means a pane rather than the status bar mark: the summary identifies the mark on line 2 of the prime's status bar, and the check spawns no pane, so the change meets the Goal's "close or shrink" idea. A failed check keeps one short item and `lym beam status` names the suite log, so no finding.
- Missing doc changes elsewhere: the affected-set lens found none, and `internal/reedengine/doc.go` states the empty-text unset already.
