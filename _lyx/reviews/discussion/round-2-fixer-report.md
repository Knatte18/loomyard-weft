# Fixer report — discussion round 2

## Changes

- **F1** (MEDIUM, consistency), fixed as proposed in `_lyx/discussion/decision-record.md` D4.
  The Decision no longer says an absent `lenses.yaml` "starts from the template (or the prime's copy under seeding)"; it says a target started from the template takes burler's mappings.
  A new Decision sentence says an absent target seeded from the prime's copy keeps whatever `lenses` or `fans` that copy holds, and burler's are dropped as kept-target.
  The one-time-window paragraph now names a prime worktree-layer `lenses.yaml` written before the first walk after deploy, for example by `lyx config lenses --set` at the worktree layer, and gives it the same way forward.
  A new acceptance criterion covers the seeded prime-copy case.
- **F2** (LOW, decision), fixed as proposed in D2's Bound.
  A new sentence says `--set` takes `lenses.<name>` as a string scalar and `fans.<name>` as a YAML flow list of lens names that replaces the fan whole.
  It also says a key with a further dotted segment under either path is refused as unknown by `Set`'s unknown-key check.
- From the self-check (fix-consistency):
  - D4's Bound sentence "Only a target started from the template in the same pass takes burler's mappings" now reads that a target takes a burler key's mapping only when it was started from the template in the same pass or does not hold that key, which matches `takeKey` (`held >= 0 && !target.fromTemplate`).
  - The first migration acceptance criterion now says it applies when there is no prime copy of `lenses.yaml` to seed from.

## Self-check

- fix-consistency, MEDIUM: the "Only a target started from the template" sentence contradicted the seeded-copy sentence for a key the copy lacks. Fixed as proposed.
- fix-consistency, LOW: the first migration criterion did not say it assumes no prime copy to seed from. Fixed as proposed.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
