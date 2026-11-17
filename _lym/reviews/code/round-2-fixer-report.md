---
dispositions:
  - id: F15
    disposition: fixed
---

## Changes

- F15: `internal/beamcli/cli.go` declares the exported `NoPrimeClaimFlag = "no-prime-claim"` and `startCmd` uses it in `BoolVar` and `MarkHidden`.
  `internal/machinecli/start.go` drops `noPrimeClaimFlag`, imports `internal/beamcli` and passes `"--" + beamcli.NoPrimeClaimFlag` to the by-slug child.
  `internal/beamcli/start_test.go` looks the flag up with `Lookup(NoPrimeClaimFlag)`.
  The literal rows of `wantBeamStarts` in `internal/machinecli/start_test.go` stay, pinning the child's command line.
  Committed as `4e21c7bd`; the tier-0 gate passed for beamcli and machinecli.
- Self-check follow-up: `pattern/PATTERN-cli-cobra.md` lists `beamcli` among the packages `machinecli` calls.

## Self-check

- fix-proposal, LOW: `PATTERN-cli-cobra.md` machinecli dependency list omits the new `beamcli` import — fixed as proposed.
- fix-code-correctness: returned a report with no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
