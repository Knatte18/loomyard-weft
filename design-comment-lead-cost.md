# The comment check costs more than the plan review

`comment-lead-cost` · Note · enhancement, lint, burler

## What was seen

`tools/tokencount -since 2026-11-03` over 359 tapestry and darn runs (2026-11-18):
the `comment-lead` role cost $150.66 (Sonnet) and its forks $184.25 (about 20 500 Haiku messages), $334.91 together, 8 % of all run spend.
That is more than the whole Plan review of the tapestry runs in the same window.

## Idea

Find what drives the cost: how many forks a run spawns, what each reads, and whether it rereads files or the whole diff per comment.
Then cut it, for example by checking only the comments the diff adds or changes, in one session without per-comment forks,
or by leaving the check to the deterministic comment lint where that finds the same.
