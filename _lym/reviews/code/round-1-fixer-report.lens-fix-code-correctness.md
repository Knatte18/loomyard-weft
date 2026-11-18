No findings.

I traced each changed function with a concrete input and a failure of each call it makes:
- `landUnderLock`, first read fails: it halts with `ownerHoldsReadReason(err)`, the same text and prefix as before.
- `landUnderLock`, second read fails (after `ownerChecks` returns nil): it halts with the same reason. A true first answer followed by a false second one reaches `landCheck` and `syncParent` as false, which is the old per-check behaviour.
- `Finalize.Call`: a parse failure is wrapped as `landingshed: Finalize: parse change description: %w`; a `completeSummary` failure as `landingshed: Finalize: %w`.
- `PRReview.pullRequestText`: a parse failure is wrapped as `read the change description: %w`; a `completeSummary` error is returned bare.
- `ParentAlign.readAttempts`: a `RowEntry` error returns before any `moveScratchAttempts` call, in the old order and text.
- `Finalize.readAttempts`: it moves the scratch record first, then resolves the entry, then calls `readRowAttempts`.
- New `OptionalSeamsLeftUnsetRunTheRow` row: a renamed or dropped name in `optionalPullRequestSeams` makes `checkWiring` return an error, so the row fails with the defect in place.

Observation, not a finding: the fix adds no test for F2's stale-answer case.

Further tool calls: none.
