# fabricengine: harden the one-repository hub's seed push and layout read

Hardens the one-repository hub layout in `internal/fabricengine`.
Live trials on a one-repository sandbox hub showed gaps in three paths, and each is closed here.

## What changes

**Seed push that ends without origin's answer.**
A `lym fabric clone --single-repo` push killed or cut off mid-flight can still land on origin, because origin's receive-pack applies the update after its client is gone.
The old rollback removed the hub and left a ref on origin that nothing could clean up.
`pushCloneSeed` now reads origin's ref before it rolls back:

- a ref at the seed's tip is withdrawn first: a landed board is deleted, then the records branch returns to the tip the clone observed;
- a ref origin does not hold at that tip yet keeps the hub and returns an error naming the refs origin may hold and the commands to clear them;
- a ref at any other tip belongs to another clone and is left alone.

A lost lease, a rejection, a stall origin never answered and a git that never ran are not unknown outcomes, and roll back as before.
A refused board push now says origin's hooks or rules declined that ref, since origin took the records branch beside it.
A completed seed now gives the records branch its upstream as well as the board branch.

**Layout detection.**
`originNamespaceAt` read a records store with a missing fetch refspec or a missing origin as the two-repository layout, so its board would push to origin's code branches.
A store whose prime key, origin push refspec or a branch's merge setting names the namespace is now a records store.
For such a store the read fails with the refspecs to restore, and the failure is not cached.

**Prime records branch.**
`primaryWeftBranch` now fails closed on a `lym.primeRecordsBranch` that names a branch the records store does not hold, or a value that is no records branch name, such as the board's branch.
Each refusal gives the command that sets the key and says to name the prime pair's own clone-time records branch.

The sandbox suite in `tools/sandbox/SANDBOX-FABRIC-SUITE.md` gains the scenarios that showed these, and integration tests cover a killed records push, a killed board push, the kept hub and the records-store faults.

## Design decisions

- The seed withdraws a ref only when origin shows it at the tip this seed pushed, so a ref another clone moved is never deleted.
- A kept hub is preferred to a removed one while origin still shows the lease tip, since a push origin is still applying could land after the read and leave a ref with no hub.
- `checkSeedPush` takes the pushed tip for either branch so the board delete and the records restore share one ownership check.
- A records-store claim from any of three settings is read as a claim, because each alone survives a damaged fetch refspec.
- A prime key naming another pair's records branch cannot be told from the right one; the refusal says so instead of guessing.

## To check by hand

- The archive tag of a discarded task branch in plain `refs/tags/`, seen in an earlier sandbox trial, is not addressed by this diff.
  Run the archive path on a one-repository sandbox hub (`lym harden sandbox new --single-repo`) and list origin's `refs/tags/`.
- Kill the seed push of a real remote at the records push and at the board push, then list `refs/lym/` on origin and confirm no ref of the seed remains and the hub directory is gone.

## Merge danger

**Door:** two-way
**Blast radius:** fabricengine
