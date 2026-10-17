# Support log — gate-wait-cache-cap

## Interview

A darn run holds no interview: the board entry, with its operator decision of 2026-10-17, is the statement of the change.

## Rejected alternatives

- Bounding `cli_wait_sec` by the cap instead of retiring it: a key whose every useful value is the cap or lower adds a knob with no reason to turn it.
- A new leaf package holding the cap: `gatecli` already reaches `shuttleengine`, so moving the constant buys nothing.
- A new exit code number: 75 is already the verb's own code and the stencils already treat it as no failure; a new number would churn every reader for no gain.
- Exit 0 with `slot_free: false`, as await does: a caller could not tell a skipped run from a passing go test by the exit code.
- An unbounded slot wait in `cmd/testtiming`: a stuck holder would hang the measurement.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Retire `cli_wait_sec` or bound it by the cap? Self-picked: retire (D2).
- Q2: Which exit code? Self-picked: keep 75 with an ok envelope (D3).
- Q3: Where does the cap live? Self-picked: `shuttleengine.DefaultAwaitCap`, imported (D1).
