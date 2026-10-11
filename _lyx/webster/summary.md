# Small fixes batch: six independent fixes landed

Both batches recorded `done`, and each card committed on its own.

## Batch 01-darn-board-get-json

Head `e666a2eb2`, covering cards 1 to 5:

- The darn and landing describe stencils read the board entry with the JSON `board get` form.
- The final-summary spec was renamed, embedded and registered so it deploys under the specs dir's `loom/` family, with every reference moved.
- The batten paragraph in `docs/overview.md` names `loom`, `darn` or empty as the accepted recipes.
- The terminal-opened marker is cleared where batten seeds a fresh status, never on a child re-spawn.
- `Migrate` hooks for shuttle, orch and loom raise a below-floor key to its floor and report the rewrite.

Reported deviation: `internal/orchengine/seam_enforcement_test.go`, whose import allowlist gained `yamlengine` because orchengine now imports it.
Informational warnings: `internal/yamlengine/floor_test.go` (`TestRaiseIntFloor`) and the orchengine allowlist were touched outside the batch's own target glyphs.

## Batch 06-fabric-unwire-apply

Head `5ed63d726`, covering cards 6 to 9:

- `lyx fabric unwire` is a read-only dry run without `--apply`.
- `lyx board prune` and `lyx board retire-legacy` are read-only dry runs without `--apply`.
- An index note prints only under the audience its annotation names, and the board group names operator.
- `WriteCarryOver` refuses before its rename and errors after it when the record changed under it.

No deviations reported.
Informational warning: `internal/clihelp/index.go` (`RenderIndex`) was touched outside the batch's own target glyphs.
The fork reported that the batch gate's package list named four `_test` directories that do not exist, so it ran the gate over the existing package directories instead.
