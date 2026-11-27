# A rejected push can force origin past a commit made by hand on GitHub

`push-over-github-hand-commit` · Note · bug, fabric, landing

Depends on: [`fabric-core-push-origin`](design-fabric-core-push-origin.md)

## Seen (board tidy, 2026-11-27)

`pr-modes-no-agent` landed with an open point in its change description: a push can still drop a hand commit made on GitHub.
For a task pair, a rejected push is retried by `resetUnnamedTip` (`internal/fabricengine/pushclassified.go`),
which can force-reset origin's code branch past the hand commit.
The change description names five other push paths that can push over such a commit.
This is read from the code and was not tested against a remote.

## Idea

A push that finds origin ahead with a commit lym did not make stops and names that commit, rather than resetting past it.
The push classification moves into the fabric core in `fabric-core-push-origin`, so the fix is made there or after it.
