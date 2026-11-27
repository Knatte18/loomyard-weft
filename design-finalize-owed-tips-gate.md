# Finalize: a parent ahead by owed landing tips loses every race and verifies under the parent lock

`finalize-owed-tips-gate` · Note · bug, landing

Imported from [issue #292](https://github.com/Knatte18/loomyard/issues/292).

## What happened

With local landing, a parent that is ahead of origin by owed landing tips always reads as moved to Finalize's locked pass.
The lock-free pass records the upstream tip (`parentMergeTip`) as the gated tip, while the parent's head is past it.
Every such landing therefore loses each of its `landing.finalize_lost_races` races, then merges and runs verify under the parent lock.
The behavior was left as it is because `gateUnlocked` and `parentMergeTip` were outside the card's targets.

## Why it is abnormal

The landing is correct, but the parent lock is held across the verify for as long as an earlier landing's push is owed.
The lock-free gate gives no benefit in that state, and other landings on the same parent wait.

## What to do

Make the lock-free pass gate against the parent's local head when that head is ahead of origin only by owed landing tips.

Provenance: task `push-retry-not-halt`, note `12-finalize-local-landing.md`.
