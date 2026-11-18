# Put the per-test coverage maps to use

`testmap-uses` · Project · feature, testmap, gate

The uses the map study ranked worth building (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`, `884424fb`).
What only the maps give is the lookup between a function and the tests that reach it, across packages; none of the uses saves test time.

Build order: the reach note first, since the other two read the maps through its reader; then failure triage; then the diff mutation check, starting with its hand trial.
The maps need no upkeep per landing for these: a stored map stays sound for days, and the packages a diff touches are mapped fresh in seconds.
