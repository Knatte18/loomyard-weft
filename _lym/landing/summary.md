# fabricengine: name a true way forward after a killed one-repository seed

A `--single-repo` clone killed with `kill -9` after its seed push leaves origin holding the whole seed.
`lym board sync` then refused with "it was never seeded onto origin" and named `lym fabric clone --single-repo ... --reset`.
The sentence is false there, and a one-URL clone onto an origin that already holds a namespaced board is refused, so the advice could loop.

## What changes

`refuseUnseededBoard` in `internal/fabricengine/pushclassified.go` now fetches and reads origin's board before it words the refusal.
The refusal is still returned with no failed-push record, so nothing retries it.

- Origin holds no board under the records namespace: the board was never seeded, and the way forward stays the `--reset` clone.
- Origin holds a board that shares history with this one: the seed landed and its clone stopped before setting the upstreams.
  The refusal prints the `git config branch.<name>.remote` and `.merge` commands for the board and the prime's records branch.
- Origin holds a board with no common history, or this board holds no commits: another clone seeded origin.
  The way forward is removing the hub and joining with `lym fabric clone <url>` without `--single-repo`.
- Origin or the local records store cannot be read, or origin's fetch fails: the refusal says so and names a rerun once it answers.

`pushCloneSeed` and `withdrawUnansweredSeedPush` in `internal/fabricengine/destroy.go` now say what origin took and what the seed withdrew.
The records-branch withdrawal after an unanswered push, a deleted board, and a restored records branch each appear in the failure, matching the declined-board message.
`withdrawUnansweredSeedPush` returns a `withdrawn` flag for this.
An unanswered board push that left no pushed tip means another clone's board stands, and the failure names a join and leaves that board alone.
`doc.go` documents both behaviours.

## Design decisions

- The comparison uses a fresh fetch and `git merge-base`, so a commit another clone pushed after the seed still counts as shared history.
- A board that shares no history is treated as another hub's and never as a candidate for `--reset`, which would replace it.
- The upstream-settings way forward is printed as commands rather than applied, since the refusal path writes nothing.

## To check by hand

- `lym fabric clone --single-repo ... --reset` and the printed `git config` commands were not driven against a killed seed.
- A board origin holds that descends from this one is worded as this hub's own board, though a clone that pushed after the seed could have made it (`descendant-origin-board-called-another-clones`).
- The test expectations spell owned branch names as literals instead of deriving them (`owned-branch-literals-in-test-expectations`).
- Some refusal branches may lack a test row (`refusal-branches-without-test-rows`).
- A failed unwithdrawn board push may still name a rerun that loops (`unwithdrawn-board-failure-names-looping-rerun`).

## Merge danger

**Door:** two-way
**Blast radius:** fabricengine
