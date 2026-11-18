# Pause holds no shared lock and works on a pair with no reed session

`pause-after-crash-fixes` · Task · bug, tender, fabric, batten, reed · running · Code-Bouncer

## 1. A paused run keeps the fabric push lock

Seen 2026-11-18: with `claude-accounts-per-worktree` paused by `lym tender pause <slug>`, dobby started `lym batten run claude-accounts-per-worktree` (pid 15436).
It took `loomyard-weft/.weft/fabric.push.lock`, then logged `tendercli: a network call is held by a pause` and waited, still holding the lock.
`lym batten run memory-cap-not-enforced` then failed twice with `push lock busy ... held by pid 15436`; the orch had to kill pid 15436.

Fix:

- A pause hold never waits while holding a shared lock:
  the held call releases the push lock before it waits and takes it again after, or the pause check runs before the lock is taken.
- Dobby starts no batten driver for a paused pair.

## 2. Pause and status fail on a pair with no reed session

Seen 2026-11-18 on Neve, two minutes after a hard power-off: `lym tender pause` failed with
`pause: list the sessions of claude-accounts-per-worktree: no reed session (2 strands persisted); run "lym reed resume" to rebuild, or "lym reed up" for a bare substrate`,
and `lym tender status` failed the same way. Meanwhile dobby had already restarted `review-result-checks`.
The orch had to run `lym reed up` in each pair's worktree before the pause went through.

Fix: a pair with no reed session has no running turn to interrupt,
so pause records its entry, reports that pair under skipped, and goes on; status reports it the same way.

## Done when

- A test shows a call held by a pause does not hold the push lock, and another pair's batten run takes it meanwhile.
- A test shows dobby starts no driver for a paused pair.
- A test shows `lym tender pause` and `lym tender status` succeed with a pair that has no reed session, naming it as skipped.
