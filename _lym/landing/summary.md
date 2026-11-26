# Name a cancel's re-run in the pending record of its removal

`lym batten cancel <slug> --remove` removes the pair through `pairteardown`, which hands the removal's remote steps to a detached teardown child.
When that child is killed, it leaves a pending-teardown record of kind `remove`.
`lym fabric status` and `lym fabric prune` listed it with the way forward ``run `lym fabric remove <slug> --remote` again``.
Once nothing of the pair remained, that command refused with `pair not found`;
the command that finished the teardown was the cancel itself.

## What changes

- `fabricengine.Topology.Remove` takes a fifth argument, `rerun`: the command that retries the removal.
  It goes into the teardown job's `Rerun` field, and from there into the pending-teardown record, as a discard's re-run already did.
  A held teardown lock's refusal names it too.
  An empty `rerun` keeps today's `lym fabric remove <slug>` wording.
- `rerunWayForward` names the job's `Rerun` for a removal that carries one.
  `recordsArchiveWayForward` drops its "when origin still holds a branch of the pair" condition for such a removal,
  since the caller's command finishes whether or not origin still holds the pair.
- `pairteardown.Request` gains `Rerun`, passed to `Remove`.
  `battencli`'s `removePairAndBranch` sets it to `lym batten cancel <slug> --remove`.
- The `pairteardown` chokepoint guard counts five arguments for `Remove`.
  Every test call site passes `""`.

## Design decisions

- The board entry also asked to name the cancel whenever a record's pair is gone.
  The record now carries its caller's command, so a record a cancel wrote names the cancel whether or not the pair is gone.
  A record written by `lym fabric remove` still names that command:
  it may belong to a slug with no batten run, which a cancel would refuse,
  and telling a gone pair from one origin still holds would cost `lym fabric status` a network call per record.
- `Remove` takes the re-run as an argument rather than through a second method, so no production path calls a test-only variant.

## Tests

- `TestPendingTeardowns` gains a row: a removal record carrying `lym batten cancel cancelled --remove` lists that command as its re-run.
  It fails on the merge base, which names `lym fabric remove cancelled --remote`.
- `TestRemove_PendingRecordNamesTheRerunItsCallerGave` (tier 1) removes a pair with an unpushed records commit and a killed child.
  It checks that the record names the hand push of the archive tag, then the cancel, with no condition on origin.
- The cancel test checks that the removal request carries the cancel's command.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
