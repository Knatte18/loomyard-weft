# A beam run is seeded at Preflight, a row its pawl recipe lacks

`pawl-starts-at-preflight` · Task · bug, pawl, preflight · done

## Seen (operator, 2026-10-30)

A beam made under the Models hub stopped at once with `current_producer "Preflight" in "_lym/shed/loomyard-setup/status.json" names no producer in the list`.
The run's `seed.json` named the recipe `pawl`, but its status file was seeded at `Preflight`: `loomshed.Seed` in `internal/loomshed/seed.go` always sets `CurrentProducer` to `Preflight`, and the bootstrap shared by every inner recipe calls it (`internal/loomcli/sharedbootstrap.go`).
Loom and darn both start with the Preflight row, and the inner shed of a pawl run expects the same start, but the pawl recipe's first row is Beam-Up and it has no Preflight row.
The error text says "the producer list has changed", which is wrong here: nothing was removed.
The Models orch moved the run on with a `goto` to Beam-Up.

## Design (operator, 2026-10-30)

- Pawl starts with a Preflight row, as loom and darn do, so the seed and the recipe agree.
- The Preflight row is the general one in `internal/preflightshed`, which any producer list may name.
