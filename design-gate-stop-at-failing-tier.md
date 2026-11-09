# The verify gate runs slower tiers after a tier already failed

`gate-stop-at-failing-tier` · Note · bug, gate, darn

## Seen (operator and orch, 2026-11-09)

The Darn-Write verify gate of `arras-recipe-perch` failed tier 0 at 15:43:27 with 56 failures, and the same gate started tier 1 at once and ran it for 11 minutes before it failed on two tests, then ran tier 2.
The first gate attempt took about 17 minutes; the tier 0 failure alone was known after 2.4 minutes.

## Idea

When a tier fails, stop the gate there and report that tier's failures, so the fixer works on them before a slower tier runs.
Check first whether running every tier after a failure is deliberate, for instance to hand the fixer every failure at once, and whether it differs between `lym gate test --upto` and the Darn-Write gate.
If a full report is wanted, make it a separate choice, and let the verify gate default to stopping at the first failing tier.
