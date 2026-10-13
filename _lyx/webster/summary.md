# Judge cap call: the Bouncer settles a spent budget from fix tags

Both batches reported `done`.

## Batch 01-ledger-fix-tag (cards 1–5, head `184b6ff9a`)

The judge now tags every open ledger entry mechanical or design-wrong, and the ledger parser keeps the tag.
Hard-cap became a third cause in the escalation record and the circling decision file.
`BouncerConfig.HardCap` makes a recorded continue at the cap re-escalate as hard-cap; escalations there offer only accept, and continues past the soft cap are exempt.
`shedrecipe.Env.RowHardMaxBounces` reaches each Bouncer row's `HardCap`.
The Bouncer settles a spent budget by the soft-cap rule, including auto-accept's replay and blocking paths.
No deviations from the declared file-ops.
Informational scope warnings named `Bouncer.blockingSendsToRule`, `Bouncer.openBlocking`, `softCapPath`, `softCapRuleRow`, `runSoftCapRuleRow` and `contracts/stencils/stencils.go`.

## Batch 06-loom-hard-cap-config (cards 6–9, head `0e1daed8f`)

`loom.yaml` gained hard-cap keys with load refusals and per-layer routes, wired into the loom Bouncer rows.
`darn.yaml` gained a hard cap for the Darn-Review perch, with its load refusals and wiring.
`lyx loom circling continue` refuses at or past the perch's hard cap.
The arbiter's review rule now varies per escalation cause, and the orch's escalation guidance and small-task advice were updated.

Reported deviation: `internal/loomengine/caps.go` is a new file holding `Perch`, `PerchCaps`, `Config.Caps` and `CapRoute`; the card named these symbols but no file.
Informational scope warnings named `hardCapSetCommand` and `softCapSetCommand` in `internal/darnengine/config.go` and `RecordCirclingDecision` in `internal/shedadapters/circling.go`.
