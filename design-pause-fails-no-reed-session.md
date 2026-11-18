# A hub pause after a crash fails on a pair with no reed session

`pause-fails-no-reed-session` · Note · bug, tender, reed

## What was seen (2026-11-18, Neve, two minutes after a hard power-off)

`lym tender pause` failed with
`pause: list the sessions of claude-accounts-per-worktree: no reed session (2 strands persisted); run "lym reed resume" to rebuild, or "lym reed up" for a bare substrate`,
and `lym tender status` failed the same way. Meanwhile dobby had already restarted `review-result-checks`.
The orch had to run `lym reed up` in each pair's worktree before the pause went through.

## Idea

A pause must work exactly when the machine just came back: a pair with no reed session has no running turn to interrupt,
so pause records its entry, reports that pair under skipped, and goes on; status reports it the same way.
