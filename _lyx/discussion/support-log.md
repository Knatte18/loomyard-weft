# Support log — dobby-skip-done-runs

## Interview

A darn run holds no interview; the board entry's operator fix is the statement of the change.

## Rejected alternatives

- Skipping done runs in beamcli's `parentRuns`: it also feeds `dobbyLandings`, so the landing count would lose done runs' history, which the entry keeps.
- Skipping done runs only in `dobbyRuns`: correct for the hub wiring, but the engine's harness could not pin it and another `Runs` seam would reintroduce the bug.
- A new state-record cleanup pass for done runs: `recordOwed` already drops unlisted slugs, so a second path adds nothing.

## Review rounds

_No rounds yet._

## Question ledger

- Do `failed` runs also stop counting? Self-pick: no; the entry names only `done`, and the doc counts halted runs on purpose.
- Does a done run with a pass in flight count? Self-pick: yes until the exit is read, since its process still runs.
