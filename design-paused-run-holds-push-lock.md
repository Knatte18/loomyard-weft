# A paused run held by its pause keeps the fabric push lock, blocking every other run

`paused-run-holds-push-lock` · Note · bug, tender, fabric, batten

## What was seen (2026-11-18)

With `claude-accounts-per-worktree` paused by `lym tender pause <slug>`, dobby started `lym batten run claude-accounts-per-worktree` (pid 15436).
It took `loomyard-weft/.weft/fabric.push.lock`, then logged `tendercli: a network call is held by a pause` and waited, still holding the lock.
`lym batten run memory-cap-not-enforced` then failed twice with `push lock busy ... held by pid 15436`; the orch had to kill pid 15436.

## Idea

A pause hold never waits while holding a shared lock: the held call releases the push lock before it waits and takes it again after,
or the pause check runs before the lock is taken.
Dobby should also not start a batten driver for a paused pair at all.
