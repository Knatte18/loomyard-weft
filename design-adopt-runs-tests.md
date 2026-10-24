# An adopt runs the test suite on the adopting machine

`adopt-runs-tests` · Note · enhancement, fabric, gate

## Seen (2026-10-24)

The prime moved from OSL-1046 back to Neve with `lyx adopt --prime`; nothing checked that the landed code builds and passes on the adopting machine.
Machines differ: Neve runs Ubuntu 26.04, OSL-1046 Ubuntu 24.04, OSL-1033 Windows 11, so code landed on one can break on another.

## Idea (operator, 2026-10-24)

An adopt runs the test suite on the adopting machine, so a break the machine change brings shows at once, not inside the next run's gate.
Note `ubuntu-2404-floor` covers the stated floor and its mechanical check.

## Open

- Which tiers: the tiers of the `finalize` gate key (`lyx config gate`, `gates.finalize`), or tier 0 and 1 only.
  A tier already green for the clean tree is carried and runs nothing.
- Whether a red suite refuses the adopt, or only reports.
- Whether it runs in the background, so the adopt returns at once.
