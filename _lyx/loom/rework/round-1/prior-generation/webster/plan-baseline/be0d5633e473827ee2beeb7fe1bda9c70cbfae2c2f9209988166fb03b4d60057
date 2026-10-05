# Card 3 — board.yaml types and labels become ordered maps

**Edit:**
- `internal/boardengine#Config`
- `internal/boardengine#Config.Outputs`
- `internal/boardengine#Config.Vocabulary`
- `internal/boardengine#LoadConfig`
- `internal/boardengine/template.yaml`
- `internal/boardengine/doc.go`
- `internal/boardengine/config_test.go`
- `internal/boardengine/template_test.go`
- `internal/boardengine/board_test.go`
- `internal/boardengine/intake_test.go`
- `internal/boardengine/boardtest/concurrency_test.go`
- `internal/boardengine/boardtest/bench_test.go`
- `internal/boardcli/intake_test.go`
- `internal/boardcli/verbs_test.go`

**Create:**
- `internal/boardengine#Label`
- `internal/boardengine#ConfigOpenMaps`

**Uses:**
- `internal/configengine#Load`

**Intent:**
Implement D1 and D2.
`Label` holds `Name` and `Description`, with JSON tags `label` and `description` for the verb in card 6.
`Config.Types` and `Config.Labels` become `[]Label`, in `board.yaml` order, decoded by `LoadConfig` from the resolved YAML node tree rather than through a Go `map`, so file order survives.
Each key accepts a mapping of name to scalar description (an empty or null description is the empty string) or, read-only, a sequence of names (each with an empty description).
Any other value, or a non-scalar description, is refused with an error naming `board.yaml` and the key.
`Config.Outputs().Types` and `Config.Vocabulary()` keep their `[]string` fields, filled with the names, so validation, intake and the README renderer's Notes order change nothing.

`ConfigOpenMaps` returns `types` and `labels`, and `LoadConfig` passes it to `configengine.Load`, which stays the strict loader.
Template: `types` holds `bug` and `enhancement` with generic one-line descriptions in block style; `labels` is an empty map; `undecided` leaves the template.
Grep for `undecided` and keep only fixtures that configure it themselves; the `lyx board list --label bug --label undecided` help example and skill line change with card 6.

Tests: map-shaped and list-shaped files both load with the expected names and descriptions in file order; a scalar `labels`, and a mapping description, are refused naming `board.yaml` and the key; `Outputs().Types` follows the map's file order; validation accepts a key of either map and refuses any other label.
Fixtures that build `Config` with `[]string` move to `[]Label`.
`internal/boardcli/verbs_test.go`'s label-filter and `--text` listing tests upsert `undecided` through a `board.yaml` that `seedCwd` writes without `labels`, so they relied on the template default this card removes; they write a `labels` entry for it themselves.

`doc.go`'s Labels section describes the two maps and the read-only list shape; the "loomyard's own area labels" paragraph stays as written, since the list shape still loads.

**ImpactSummary:** board config decode and its template; every test fixture constructing `boardengine.Config` with labels.

**Commit:** 3: board.yaml types and labels become ordered maps
