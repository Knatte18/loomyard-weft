# Round 3 fixer report — discussion

## Changes

- **F1** (fixed as proposed): the second `## Open risks` bullet now limits the rewrite-and-rebaseline remedy to the `--tags` line, which D2 refuses.
  A new sentence says that an in-flight gate call without `--wait` is not one `lyx gate test --wait` call.
  So the runner takes its single slot, the nested call inherits that slot (D6, D8), the call waits without the keep-cache-warm cap and runs under the runner's 10-minute timeout, and it does not fail on a busy hub.
- **F2** (fixed, with the PATTERN wording changed differently): a new `## Acceptance criteria` bullet follows the `cmd/testtiming` bullet.
  It pins each non-tier holder's child GOFLAGS: `-trimpath`, plus `-p=<tier0's parallel>` only when the hub declares `tier0`, and `cmd/testtiming` outside a hub using the CPU count with no slot.
  - I did not use the proposed `PATTERN-gate-slot` wording, "setting its own `-trimpath` and `-p` Go flags". It makes `-p` unconditional, while D13 (`decision-record.md:261`) and the new criterion set no `-p` when the hub has no `tier0`.
    The bullet now reads "setting its own `-trimpath` Go flag, plus `-p` from `tier0`'s parallel when the hub declares `tier0`".
  - The "Goal-adjacent mirror" the fix names does not exist. A grep for "own Go flags" finds only the `## PATTERN changes` bullet, so that bullet is the only text changed.
- **F3** (fixed as proposed): the first sentence of `## Goal` now puts "apart from D13's non-tier slot holders" before "goes through `lyx gate test`", so "which runs numbered tiers" attaches to `lyx gate test`.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-way-forward: no findings.
- fix-proposal, MEDIUM: the proposed `PATTERN-gate-slot` wording made `-p` unconditional and contradicted D13's no-`-p`-without-`tier0` rule. Fixed with my own change, recorded under F2 above.

## Disputed

None.

## Deferred

None.
