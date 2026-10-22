# Support log — convergence-darn-landing-commit

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Bounding the walk by the seed's `started_at` time: it compares a commit's committer time against a stamp, which clock skew between machines can break, and a partial tag with no decodable seed has no stamp.
- Bounding the walk by the merge base of the tag and the board branch: a landed pair's records are merged back into that branch, so the merge base is the tag itself or above it and bounds nothing.
- Writing a task-commit trailer on a darn run's change-description commit: that names the task tip before Merge-In and review fixes, so the squash landing's tree rarely matches it; the exact join would then be attempted and fail, leaving the run unmapped where the time join maps it.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Should a darn run map through the exact join or the time join? Self-pick: the time join, since no darn record names the landed task tip; resolved.
- Q: Which commit marks the start of a run's own records history? Self-pick: the pair-opening origin-record commit `fabric: record parent branch for <slug>`, which pair creation always commits on a new pair; resolved.
