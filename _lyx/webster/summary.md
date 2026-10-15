# forkaudit: a repeatable Webster fork read audit

The single batch, `01-cli-and-output`, finished `done` at head `161d3839550703ec05607f45fcb9048edcdcbb58`.
It covered the plan's six cards, which built `tools/forkaudit/`: the script, its jq filters, the test fixtures and the integration test package.

The digest's deviation list, from the glyph scope guard, holds one file: `tools/forkaudit/forkaudit_integration_test.go`.
That file is the batch's own test package; the guard flagged it as changed outside the declared targets, and it is informational only.
The batch ended with no warnings.
