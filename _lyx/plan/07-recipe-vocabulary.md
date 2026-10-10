# Card 7 — recipe-vocabulary

**Create:**
- `internal/shedrun#RecipeDarn`
- `internal/segmentcolor#Darn`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun/seed_test.go`
- `internal/shedcli#recipes`
- `internal/shedcli/doc.go`
- `internal/shedcli/table_test.go`
- `internal/segmentcolor#Segments`
- `internal/segmentcolor/segmentcolor_test.go`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine/config_test.go`

**Intent:** `darn` joins the closed vocabularies a recipe is named in: `shedrun.RecipeNames` returns `batten`, `darn`, `loom`, so `ValidateRecipe` and `WriteSeed` admit it (`PATTERN-shed-run-directory`);
the `lyx shed` table gains a `darn` row armed by `loomcli.ArmAt` with all five generic verbs and `loomcli.BootstrapVerb`, no seed-location rule, so the key set keeps matching the vocabulary (`PATTERN-shed-verb-set`), and the shedcli doc names the three recipes.
`segmentcolor.Segments` gains `Darn`, the launch-spec segment the darn writer carries (`PATTERN-spawn-color`), and both reed templates give `segment_colors.darn` the palette color `red`, the one color the seven segments leave free.
Tests: the shedrun vocabulary tests and the shedcli key-set and entries tests gain the `darn` rows, the segment closed-set test gains `darn`, and reed's config test map gains `darn: red`.
Until the later cards land, a `darn` seed arms loom's own graph through the same entry; the arming is per recipe from the loom-arms-darn card on.

**ImpactSummary:** Three closed sets each gain one member; every pinning test is on this card, and no verb body changes.
