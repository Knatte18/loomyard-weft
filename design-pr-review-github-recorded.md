# PR-Review session tests drive pullrequest.Client over a recorded GitHub stand-in instead of prkit.API

`pr-review-github-recorded` · Task · enhancement, testkit, landing · draft

Depends on: [`parent-align-fixer-own-push`](design-parent-align-fixer-own-push.md)

## Seen

The Code-Review of `pr-session-mail-stand-ins` (round 1, finding F2) found that its new tier1 PR-Review mail test adds uses of `prkit.API`,
whose stand-in inventory verdict is "goes".
The tree has no recorded GitHub stand-in to drive `pullrequest.Client` over, so the parent accepted the use for that task.

## Idea

Give testkit a recorded GitHub stand-in, in the manner of the recorded tmux kit, that `pullrequest.Client` runs over,
and move the PR-Review session tests off `prkit.API` onto it.
