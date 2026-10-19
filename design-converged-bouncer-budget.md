# A stop after a converged review verdict spends no review budget, and its way forward keeps the verdict

`converged-bouncer-budget` · Note · bug, shed, loom, burler

## Seen (2026-10-19, run `ubuntu-2404-support`)

The Plan review round converged (`round-1-bouncer-verdict.md`: `CONVERGED`).
Plan-Bouncer then failed on its own commit of the approved plan (a pathspec bug, fixed on main as `7a3396d05`).
Each retry of that commit counted against the Plan-Review segment's bounce budget, so once the fix was deployed the run blocked with "bounce budget exhausted for Plan-Bouncer", whose way forward named `lyx shed goto self --to Plan-Bouncer`.
The orch took that goto; it gave the whole segment a fresh budget, the converged round no longer counted, and the Bouncer started a new review round over an approved plan.

## Idea

- A stop after a converged verdict, such as a failed commit or other mechanical step of the Bouncer, does not spend the segment's review budget: retrying it re-runs only the step that failed.
- When the budget does run out at a Bouncer whose round converged, the error names `lyx loom circling accept` and `lyx loom resume`, never a goto, since a goto discards the verdict.
