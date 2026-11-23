# One-repository fabric cases no hardener round drove

`single-repo-undriven-cases` · Task · bug, fabric · running · PR-Review

## Seen

The second fabric hardener campaign (`harden-fabric-single-repo-verbs`, landed `2722a66ef`) approved round 3 with part of its focus never driven live.
`harden-fabric-undriven-cases` fixes the faults the campaign found but could not raise (O1 to O5); these cases were never tried at all:

- A discard killed mid-operation.
- A fabric step that fails with no kill.
- `lym adopt --archives` over real owner archive tags.

No regression check ran for that landing.

## Idea

Drive each case on a sandbox one-repository hub, live, after `harden-fabric-undriven-cases` lands, and file each fault found.
