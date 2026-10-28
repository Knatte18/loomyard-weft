## Changes

Each fix is its own commit; the gate was green (`lym gate test ./internal/loomshed ./internal/gatecli --tier 0 --tier 1`) after each.

- F1 (`2f5c43d8`): `outputAboveHeader` in `internal/loomshed/verifygate.go` takes the failing package and searches only that package's block, from the line after the previous `ok`, `FAIL` or `?` closing line to the package's own `FAIL\t<pkg>` line; the upward scan stops at the block start.
  Added a `TestVerifyFailureFindings` row where two packages fail a test of the same name.
- F3 (`72b4cccd`): `failuresSection` cuts an entry longer than `verifyFailuresBytes` on a rune boundary with a `[truncated]` marker; the constant's comment is updated.
  Added a row with an oversized output and a following failure, asserting the cut, the marker and the `More failures` line, plus a UTF-8 validity assertion on every row.
- F5 (`4958b604`): fixed differently.
  The proposed change asserts that nothing was announced as rerun, but `retryOnShortage` announces before the delay, so that assertion would be false.
  Added a `TestRetryOnShortage` row that cancels the call when the rerun is announced, with a one-hour delay, and asserts one attempt, the first result and the announcement.
  The log-prefix assertion applies only when the rerun ran.
- F4 (`d264039a`): the build scenario sets the inherited MSBuild variables and asserts the build step's values; a new scenario row asserts a tier's own `env` wins over the gate's; `TestGateTest_UnsetDirFollowsTheAnchor` gains a tier with `dir: sub` asserted to run in `sub` under the worktree root.
  The stand-in already recorded the MSBuild variables in every mode, so no recording change was needed.
- F2 (`d01ce260`): `newCommand` takes the shortage retry delay, `Command()` passes the default and the test helper passes 0; the stand-in gains `helperShortageEnv` (`once` or `always`).
  Two `TestGateTest_Scenario` rows drive `runTest`: a once-failing tier ends exit 0 with the rerun announced, and a twice-failing tier keeps exit 1 and is named a resource shortage in the closing line.
  The output is read from the gate's stdout, where the helper routes its stderr.
- Self-check findings (`7e00bf2f`): `outputAboveHeader` goes on to the next block of the same package when a block holds no matching header, with a row for a package repeated in the log; the helper's two comments name the shortage delay and `helperShortageEnv`.

## Self-check

- fix-code-correctness, LOW: `outputAboveHeader` stopped at the first block of a repeated package. Fixed as proposed in `7e00bf2f`.
- fix-consistency, LOW: the comment above the helper duration constants omitted the fixed shortage delay. Fixed as proposed in `7e00bf2f`.
- fix-consistency, LOW: the `helperRoleStandIn` comment omitted the shortage branch. Fixed as proposed in `7e00bf2f`.
- fix-affected-set: no findings.
- fix-proposal: no findings; its note on F5 is the own-change mark above.
- The four lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`; they are condensed, not the forks' verbatim messages.

## Disputed

None.

## Deferred

None.
