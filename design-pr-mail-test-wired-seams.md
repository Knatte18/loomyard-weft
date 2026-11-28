# The PR-Review mail test takes its CommitRecord, BaseCommand and GateTiers seams from loomcli wiring instead of envkit stand-ins

`pr-mail-test-wired-seams` · Note · enhancement, testkit, landing

## Seen

Code-Review round 2 of `pr-session-mail-stand-ins` (finding F10, BLOCKING) was closed by an accepted decision with the finding unfixed.

## Built since

The landed test (`c17e0a5f8`, `internal/loomcli/prreview_mail_integration_test.go`) assigns `DirtyPaths`, `CommitsMissing`, `TaskHasCommits`, `PauseCovered`, `RecordChanges`, `RecordPath` and `RecordLockPath` from `wired.PullRequest`,
and its setup commits on the pair's HEAD beyond the parent branch.

## What is left

`CommitRecord`, `BaseCommand` and `GateTiers` still keep `envkit`'s hand-written answers.
The landing's description says the wired `CommitRecord` pushes to the pair's origin, a GitHub URL in this test,
and the wired gate seams drop the tier from the mail and halt reason on this fixture hub.

## Idea

Take those three from `loomcli` wiring as well, once the test setup gives the wired seams what they need.
