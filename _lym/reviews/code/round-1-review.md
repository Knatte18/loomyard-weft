---
verdict: APPROVED
---

No findings.
The diff makes the change the amended board entry asks for: the pawl recipe opens with the general `Preflight` row, `entry` is `Preflight`, `on_done` is `Beam-Up`, and the row carries no `on_stuck`, as darn's does.
`Preflight` joins `holdFreeProducers` for a beam project alone, and the docs and comments that list the pawl run's rows before Parent-Align name it.
The seed and the refusal wording stay untouched, as the record's out-of-scope list says.
The change description's claims match the diff, and the commit needs no `Pattern-Departure:` trailer.

Focus questions settled:

- `Env.Cwd` is assigned unconditionally at `internal/loomcli/wiring.go:556`, before the pawl branch at line 833, so a pawl run's `Env` carries the absolute `Cwd` the `Preflight` row's `requireAbsRoot` demands. `TestWire_BeamRecipe` wires a pawl seed and then builds its routing, which reaches that row.
- The recipe's `Preflight carries no on_stuck` paragraph gives the reason darn's header gives for the same row, and the other rows' on_stuck wording is unchanged.
- No pawl-run doc still lists the old first rows. `internal/battenshed/doc.go`, `internal/orchcli/doc.go`, `internal/pawlengine/doc.go`, `internal/beamcli` and `docs/overview.md` name only Worktree-Create, Beam-Up or the producers `pawlengine` owns.
- `RunHolds` is the sole reader of `holdFreeProducers`, and it lifts the hold for a beam project alone. `runTakesNoSlot` and the `lym loom peers` list read through it, so a non-beam entry at `Preflight` keeps its hold and a halt at `Preflight` holds nothing, as D2 bounds it.
- The diff adds no new test function. `TestRunHolds/BeamAtPreflight`, the `wantPawlRows` row and `TestPawlRouting`'s entry assertion pin the new row, and `TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` walks the recipe's rows, so it fails if the hold-free list lacks `Preflight`.

## Rejected

- The code-tests lens's LOW `design` finding that `TestRunTakesNoSlot` has no `Preflight` row: `runTakesNoSlot` has no branch of its own for the row, since it reads `RunHolds`, which `TestRunHolds/BeamAtPreflight` pins. The table already pins `Beam-Await` and `Worktree-Create` through that same path, and no row exists for `Beam-Up` either, so a `Preflight` row would duplicate their coverage.
