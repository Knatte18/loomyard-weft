# A beam cannot be made from a project whose slug already ran a motif run

`beam-after-motif` · Task · bug, batten, pawl · running · Finalize

## Seen (2026-11-03)

Task `codex-engine` ran recipe `motif` to write its Sketch, then the orch turned the entry into a project (recipe `pawl`) to split it into parts.
`lym beam create codex-engine` refused: `"codex-engine" has already completed`, naming `lym batten cleanup --apply` and deleting the old task branch.
The workaround was `lym batten cleanup --apply` from the prime, after which `lym beam create codex-engine` started.

## Expected

A motif run is the planned first step of a project split, so its done batten record does not block the project's beam.
`lym beam create` clears the slug's done record itself when that record's run was a motif run whose pair was torn down, or the motif run removes its record when it lands.
