# Rename the darn writer row to Darn-Write

`darn-write-rename` · Task · enhancement, loom, shed

Depends on: [`darn-perch`](design-darn-perch.md)

## Seen (operator, 2026-10-11)

`darn` names both the recipe and its writer row `Darn` (engine `DarnWrite`), which is confusing.
The loom recipe avoids this: its rows are named for what they do (`Discussion-Write`, `Plan-Write`, `Webster`), and no row is named `Loom`.

## Wanted

Rename the darn writer row from `Darn` to `Darn-Write`, matching the loom `-Write` rows and the engine name `DarnWrite`.
Row names are durable on-disk identities pinned against `internal/loomshed` `Name*` constants, so rename the constant with the row, and keep an in-flight darn run whose status names the old row resumable.
Update every reference: recipe edges (`PR-Gate`'s `on_stuck`, the perch rows `darn-perch` added), docs, stencils and tests.
