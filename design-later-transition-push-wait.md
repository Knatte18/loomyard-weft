# Later run transitions still wait for a records push to origin

`later-transition-push-wait` · Note · sketch, batten, fabric

Decision record: [Sketch](decision-later-transition-push-wait.md)

## What was seen

Since `71d6ee1a0` every batten transition pushes its records to origin before the run acts, at about 10 to 20 seconds each.
`transition-push-slows-rows` removes the push from the start rows only; Land, Worktree-Teardown and every other later transition still wait for it.

## Idea

Push only at the transitions a takeover reads from, batch several transitions into one push, or push in the background and wait only before a row whose effect a second machine could repeat.
The design must show which transitions adopt needs pushed before the act.

## Loom rows pay it too

Skipped loom rows sat about 2 seconds apart before `71d6ee1a0` (runs of 2026-11-12 and 2026-11-17) and about 6 seconds apart after it (2026-11-19, 2026-11-21, 2026-11-22), read from the `at` times of the Drift rows in archived `status.json` files.
A darn walks about 25 rows, so the waits add a few minutes per run.
