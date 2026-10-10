# Webster Master fixes a verify failure itself instead of spawning the fixer fork

`master-self-fix` · Note · bug, webster, stencil

## Seen

In `driver-context` (2026-10-10), Webster's Publish verify failed twice and passed on attempt 3.
On that attempt Master, running `sonnet[medium]`, fixed the failing assertion itself (`Edit` on `internal/stencilcli/cli_integration_test.go`, then `git add` and `git commit`, 99d75755c), although its stencil sends verify failures to one fixer fork.
The fix was correct, but the run-exit audit flagged it as a parent-write and blocked the run, which then had to revert the file, `accept-audit` and redo the fix through a fork.

## Idea

Make the rule harder to skip: state it at the point Master reads a verify failure, not only in its rules list, or give Master a verb that spawns the verify fixer so the path of least resistance is the right one.
Check whether `sonnet[medium]` as Master skips such rules more often than a stronger model.
