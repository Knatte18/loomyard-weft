# Split the long battencli integration scenario so its parts run in parallel

`battencli-split-long-tests` · Task · enhancement, batten, testkit · starting · Worktree-Create

## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/battencli` at 33 to 75 s of tier1 test time.
`TestBattenIntegration_Rows` (54 s) runs its steps one after another on one hub copy and sets the package's floor.

## Work

- Split it into groups with a hub each, so the groups run in parallel.
- Keep every assertion; drop none.
- Leave the cancel tests' structure for a later task.
- The two measured runs differed by more than a factor of two: measure the package before and after three times each, and put them in the change description.
