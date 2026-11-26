# Periodic async push of pairs whatever made the commits

`periodic-pair-push` · Note · enhancement, fabric, dobby

Imported from [issue #247](https://github.com/Knatte18/loomyard/issues/247).

## Idea

Agents should only have to commit; they should never have to think about pushing.
lym takes care of every push itself, asynchronously, without blocking an agent.

## Seen

lym already pushes asynchronously after `lym fabric commit`, but only then.
An agent that commits with plain `git commit` leaves its commits unpushed until some later lym step pushes the pair, and the push failures that follow land on whichever step happens to push next.

## Proposal

A periodic pusher in lym, for example in the dobby or its own small daemon, that pushes each live pair's code and records branches when they are ahead of origin, however the commits were made.

- Periodic and not aggressive: a fixed interval of minutes, not a push per commit.
- Never blocks an agent or a row; a failed push is retried at the next period, with backoff, and reported once rather than per attempt.
- Rows that need origin current, such as Finalize, still push synchronously as now.
