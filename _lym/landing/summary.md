# Name the cancel as the re-run in its removal's pending record

`lym batten cancel <slug> --remove` hands its pair's remote teardown to a detached child.
If that child was killed, `lym fabric status` told the operator to run `lym fabric remove <slug> --remote`.
Once the pair was gone, that command answered `pair not found`, and the cancel was what finished the teardown.

A removal started by a cancel now records the cancel as its re-run, and running it finishes the teardown:

- The way forward of the pending record, of a failed remote step and of a held teardown lock names `lym batten cancel <slug> --remove`.
- That cancel no longer refuses with "nothing of the slug is left to cancel" while a pending teardown record names it.
- When nothing of the pair remains, locally or on origin, the cancel pushes any of the slug's archive tags that origin lacks and drops the record.
  A failed push keeps the record and names the cancel to run again.

Removals whose caller names no re-run still point at `lym fabric remove <slug> --remote`.

The change also stops the scheduler in a `dobbyengine` landing-notice test before the test lists the orch,
so that a delivery worker still probing cannot type the notice into the pane ahead of the test's read.

## Design decisions

- `Topology.Remove` and `pairteardown.Request` take the caller's re-run command; empty keeps the old wording.
  A caller that passes one must be able to finish the teardown once nothing of the pair remains,
  because the record names its command with no "when origin still holds a branch" condition.
  The enforcement scan for stray `Remove` calls now matches five arguments.
- The re-run is chosen by who wrote the record, not by whether the pair is gone.
  A record written by `lym fabric remove` or another caller without a re-run keeps naming `lym fabric remove` even after its pair is gone.
  This is deliberate, although the board entry also suggested the pair-gone test:
  the cancel counts only records that name it, and testing for a gone pair would cost `lym fabric status` a network call per record.
- Only the cancel calls the new `Topology.SettleGonePairTeardown`.
  It holds the slug's teardown lock and leaves the record alone while any part of the pair is left.

## To check by hand

- The reported scenario was not driven live: kill the `lym fabric remote-teardown` child of a `lym batten cancel <slug> --remove`
  while it stalls on an origin branch delete, then follow the way forward `lym fabric status` prints.
  The cancel's integration test writes the pending record by hand instead.
- A record left by a killed cancel before this change holds no re-run, so it still names `lym fabric remove`
  and does not by itself keep the cancel from refusing with nothing left.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
