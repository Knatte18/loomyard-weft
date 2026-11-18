# P3 Put the per-test coverage maps to use

Rendered 2026-11-18 18:06 UTC; every age below is as of then.
Under: [Board](README.md).

[`testmap-uses`](design-testmap-uses.md) · Age 2m · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

The uses the map study ranked worth building (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, `884424fb`).
What only the maps give is the lookup between a function and the tests that reach it, across packages; none of the uses saves test time.

Build order: the reach note first, since the other two read the maps through its reader; then failure triage; then the diff mutation check, starting with its hand trial.
The maps need no upkeep per landing for these: a stored map stays sound for days, and the packages a diff touches are mapped fresh in seconds.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F4 |  | **Reach note for a diff: the tests reaching each changed function, and changed functions nothing reaches**<br>• A reader of the coverage maps feeds the code review and a read verb for writers; uses 1 and 3 of the map study read through it | [`testmap-reach-note`](design-testmap-reach-note.md) | 8m<br>2m | testmap, gate, burler |
| F5 |  | **Mutation check of the code a diff changes, survivors to the code review**<br>• Mutate only the blocks a diff adds or changes, run only the tests covering each, report surviving mutants; start with a hand trial over about twenty landed diffs | [`testmap-diff-mutation`](design-testmap-diff-mutation.md) | 8m<br>2m | testmap, gate, burler |
| F6 |  | **Failure triage: does a failing test reach what the run changed**<br>• The gate failure report and the fixer brief say which changed functions a failing test reaches, or that it reaches none, so a not-my-fault failure is seen at once | [`testmap-failure-triage`](design-testmap-failure-triage.md) | 8m<br>2m | testmap, gate, darn |
