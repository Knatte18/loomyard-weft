# Card gate runs tier0 only and misses tier1 breakage a card causes

`card-gate-misses-tier1` · Note · bug, webster, gate

Imported from [issue #896](https://github.com/Knatte18/loomyard/issues/896).

A card that made `PaneMail` persist on every strand broke a tier1 `creelcli` test.
The card gate runs tier0 only, so it did not run that test; the later batch gate found the breakage.
The failure surfaced a card late, away from the change that caused it.

This is abnormal: a card's gate can pass while its own change breaks a higher-tier test, so the fix lands outside the card that introduced it.
Consider letting the card gate run the tier1 tests of packages the card's diff touches, or flagging in the card result that higher tiers were not run.

Provenance: task `creel-hook-post`, note `07-shuttle-idle-no-turn.md`.
