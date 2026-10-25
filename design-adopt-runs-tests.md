# An adopt runs the test suite on the adopting machine

`adopt-runs-tests` · Task · enhancement, fabric, gate · running · Describe

## Seen (2026-10-24)

The prime moved from OSL-1046 back to Neve with `lyx adopt --prime`; nothing checked that the landed code builds and passes on the adopting machine.
Machines differ: Neve runs Ubuntu 26.04, OSL-1046 Ubuntu 24.04, OSL-1033 Windows 11, so code landed on one can break on another.

## Idea (operator, 2026-10-24)

An adopt runs the test suite on the adopting machine, so a break the machine change brings shows at once, not inside the next run's gate.

## Operator decisions (2026-10-25)

- This replaces the note `ubuntu-2404-floor`: no PATTERN entry for an Ubuntu floor and no CI job.
  Adopting onto OSL-1046 (Ubuntu 24.04) runs the full suite there, and what fails is fixed there and then, on the machine that shows the break.
- The full suite runs once per adoption session on a machine, not once per adopted worktree: adopting the prime and two beams together runs it once.

## Open

- What "full suite" maps to: every tier, or the tiers of the `finalize` gate key (`lyx config gate`, `gates.finalize`).
  A tier already green for the clean tree is carried and runs nothing.
- What marks an adoption session: one `lyx adopt` call, or every adopt on the machine until the next commit lands.
- Whether a red suite refuses the adopt, or only reports.
- Whether it runs in the background, so the adopt returns at once.
