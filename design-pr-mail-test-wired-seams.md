# The PR-Review mail test takes every git, pause and record seam from loomcli wiring instead of envkit stand-ins

`pr-mail-test-wired-seams` · Note · enhancement, testkit, landing

## Seen

Code-Review round 2 of `pr-session-mail-stand-ins` (finding F10, BLOCKING) was closed by an accepted decision with the finding unfixed.
`prReviewMailDeps` takes only `Mode`, `Origin` and `TaskHead` from `wired.PullRequest`.
It still reads envkit's hand-written `DirtyPaths` (nil), `CommitsMissing` (nil), `TaskHasCommits` (true), `PauseCovered` (false) and `RecordChanges` (nil channel),
standing in for verifytree, fabricengine, the pause record and the record watch over a real pair.

## Idea

Assign those seams from `wired.PullRequest` as well, and make the state a wired seam needs through gitkit or fabric in the test setup,
such as a commit on the pair's HEAD beyond the parent branch for `TaskHasCommits`.
Keep `RecordPath` and `RecordLockPath` in step with the wired record paths.
