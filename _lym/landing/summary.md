# A board with no upstream reads origin before naming its way forward

A `--single-repo` clone sets the board's and the records branch's upstreams only after both seed pushes land.
A clone killed between the last push and those settings leaves origin holding the whole seed and a local board with no upstream.
`refuseUnseededBoard` (`internal/fabricengine/pushclassified.go`) inferred "never seeded onto origin" from the missing upstream alone and named a `--reset` clone.
A `--single-repo` clone onto an origin that already holds a board is refused, so that way forward looped.

## What changes

`refuseUnseededBoard` now reads origin's board ref with one `ls-remote` before it refuses, and names the way forward origin's state calls for:

- **Origin holds no board:** the board was never seeded; the way forward stays the `--reset` clone.
- **Origin holds this board's tip or an ancestor of it:** the seed landed; the way forward names the four `git config` commands that set the board's and the prime records branch's `branch.<b>.remote` and `branch.<b>.merge`, then a rerun.
- **Origin holds another board:** another clone seeded origin; the way forward is to remove this hub and join with `lym fabric clone <url>` without `--single-repo`.
- **Origin cannot be read:** the refusal says so and asks for a rerun once origin answers.

The refusal still writes no failed-push record and pushes nothing.

A seed push whose client died before origin answered, and which origin had applied, is withdrawn as before.
Its error now says so: for the records branch, that origin took it and the seed withdrew it;
for the board, that origin took the records branch and the board, the seed deleted the board and returns the records branch.
`withdrawUnansweredSeedPush` returns whether it withdrew a ref so `pushCloneSeed` can word the cause.

## Design decisions

- The seed-landed case accepts origin's tip as an ancestor of the local board, not only equal to it, since board commits made locally after the killed seed leave origin behind without making it another hub's board.
- The way forward is a set of `git config` commands rather than fabric setting the upstreams itself: the board entry asks for a truthful message, and adopting origin's refs silently would be a new behavior.
- The `--reset` path was not driven by hand; the second hardener campaign (`harden-fabric-single-repo-verbs`) drives these verbs.

## Tests

- `TestCloneSingleRepo_FailedSeedPushLeavesNothing/BoardWithNoUpstreamIsNeverPushed` replaces the single never-seeded subtest with a table over the origin states no board, this hub's seed and another board.
  Each row asserts the message, that origin's records namespace is unchanged and that no failed-push record is written.
  The seed row then sets the named upstreams and asserts the push succeeds, which reaches the way forward.
- `TestCloneSingleRepo_SeedPushKilledBeforeOriginsAnswer`'s two landed rows assert the withdrawal wording.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
