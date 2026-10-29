# Fixer report — discussion round 3

## Changes

- F1, LOW, fixed differently.
  The premise holds: `Reading.Ack` (`internal/creelengine/verbs.go`) returns at the first failing journal write, and `hook-read`'s `writeBlock` handler prints before the client acknowledges.
  D6 now carries a rule for an acknowledgement whose journal write fails: a handed-out echo is dropped at that acknowledgement, since `hook-read` printed it already;
  on a ring, the echo keeps the lines of the ids the acknowledgement passed before the failing write, and nothing for the ids D3 released, which reach the operator when a later claim hands them out.
  The D6 acceptance bullet checks both cases, and the support log records the round.
  Two parts of the proposal were not made as written, both after the self-check below:
  the ring echo covers the ids passed before the failing write, journaled or skipped, not only the ids journaled read, because `Reading.Ack` skips an id that left the holder without journaling it, and D3's release cannot hand that id out again;
  and D5's `ack-failed` lists the ids of the echo the claim handed out, which D6 drops, and none on a ring, not the echo ids under the whole D6 rule, because D5's field is the ids of the echo a claim hands out and a ring hands out none.

## Self-check

- fix-consistency, LOW: D5's `ack-failed` clause read as listing a ring's kept echo, which contradicts D5's field of handed-out echo ids. Fixed: the clause lists the dropped echo's ids, and none on a ring.
- fix-proposal, LOW: an id `Reading.Ack` skips before the failing write was shown to the model but was neither journaled nor handed out again, so the ring echo left it out. Fixed: the echo keeps every id passed before the failing write, journaled or skipped, in D6 and its acceptance bullet.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
