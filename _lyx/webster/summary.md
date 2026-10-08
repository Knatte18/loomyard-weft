# PATTERN-shuttle-stop names orchcli's orch-strand removal as an exception

Batch 11 (`shuttle-stop-orch-exception`) was the only batch and recorded `done` at `bde81b4e13bb2256d566315fa87566722811f551`.
It rewrote the `PATTERN-shuttle-stop` entry in `PATTERN.md` so that both `loomcli`'s removal of the loom driver's strand and `orchcli`'s removal of the orch strand are named as exceptions.
No Go code changed.

The fork did not use the card's exact wording.
That wording made the entry line 409 characters, over the 320-character cap that `TestOverview_RealFilePassesCheck` enforces.
The fork shortened the entry to 315 characters, saying both removals are exempt because no attach probe or Wait reads their records.
The record-batch digest reported no deviations from the declared file-ops and no warnings.
