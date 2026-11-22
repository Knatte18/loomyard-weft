# Later run transitions still wait for a records push to origin

`later-transition-push-wait` · Task · enhancement, batten, fabric · draft

Depends on: [`transition-push-slows-rows`](design-transition-push-slows-rows.md)

## What was seen

Since `71d6ee1a0` every batten transition pushes its records to origin before the run acts, at about 10 to 20 seconds each.
`transition-push-slows-rows` removes the push from the start rows only; Land, Worktree-Teardown and every other later transition still wait for it.

## Idea

Push only at the transitions a takeover reads from, batch several transitions into one push, or push in the background and wait only before a row whose effect a second machine could repeat.
The design must show which transitions adopt needs pushed before the act.
