# Code changed after Drift-Verify lands with no compare

`drift-after-review` · Task · bug, drift, loom · running · PR-Review

Code the review rows change after Drift-Verify lands with no compare, and once the run has passed Drift-Verify no Drift verb can compare it again.

## Seen

On the `cs` hub (2026-11-03), the darn recipe ran `Drift-Verify → Code-Review → Describe → PR-Review → Finalize` for `report-significant-digits`, a change to `ResultFormatter` that every regression case renders.

- Drift-Verify compared at 17:37–17:38: five cases drifted on the report line, the agent accepted each with the values it saw, the parent ruled on them, and the row ended `done` at 17:38:55.
- At 17:42:29 Code-Burler-Fix committed `4b31b22`, which rewrote `ResultFormatter`'s formatting path (fixed-point text, rounding for tiny values, negatives).
- No compare ran after it: the newest compare log is 17:38:52; Describe, PR-Review and Finalize run gates, not Drift.
  The accepted values in the scope record describe code the PR no longer holds.
- The reviewer of the hub orch asked for a fresh compare before approving.
  `lym drift compare` refuses outside Drift-Verify ("It runs at Drift-Verify only") and skips accepted tests, so it could check neither the re-written code nor the accepted cases' new values; the compare had to be run through the harness by hand.

The same holds in tapestry, whose tail reads `Parent-Align → Drift-Verify → Code-Review → …`.

## Expected

- When a row after Drift-Verify commits a change to the code the scope reaches, the run compares again before Describe: either the recipe re-enters Drift-Verify, or a compare row after Code-Review checks the accepted tests against the values their acceptances name.
- An accepted test whose current output differs from what its acceptance saw counts as drift again.
