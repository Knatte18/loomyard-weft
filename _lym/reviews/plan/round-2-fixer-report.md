# Round 2 fixer-report — plan

Every finding's premise held against the code; all eight are fixed as proposed.
`lym loom validate-plan` reports `ok: true` with informational findings only, after the fixes and again after the self-check's changes.

## Changes

- **F1** (cards 19, 22): each card gains an `**Edit:**` section listing `internal/loomshed#loomshedAllowedImports`.
  Card 19's Intent adds `internal/raddleparser` to the allowlist with its comment line; card 22's adds `internal/boardengine`.
  `go list -deps ./internal/boardengine` holds no loomshed import, so there is no cycle.
- **F2** (card 25): Create gains `plan:internal/pawlengine#HandOffMarkerName` (`"handoff-sent"`, beside `AwaitMarkerName`); Edit gains `internal/hubgeom#PawlPaths` and `internal/hubgeom#TestDobbyGeometry`; Uses gains `internal/hubgeom/doc.go`.
  The Intent says `PawlPaths` sets `HandOffMarker` under the beam's `.lym/beam/` directory, with its doc comment dropping the count of paths ("All four"); `TestDobbyGeometry`'s directory assertion gains the marker; `pawlDeps` fills only the `HandOff` seam, and `TestWire_BeamRecipe`'s existing comparison pins the path.
  ImpactSummary names the `PawlPaths` change.
- **F3** (card 21): `RaddleTable` sets `Optional: []string{"return_note"}`, with the reason; `TestRaddleTable` gains a first-session row (blank `return_note` passes `Validate`) and a return row (filled value).
- **F4** (card 12): `Parse` finds an unknown key by also decoding each object into a `map[string]json.RawMessage` and comparing its keys against the spec's key lists, and never calls `DisallowUnknownFields` (`PATTERN-tolerant-records`). The `TestParse` unknown-key row stays.
- **F5** (card 23): `shedcli`'s `recipes` key is `shedrun.RecipeRaddle`; a new sentence makes every production site the card writes the name in (`embeddedRecipes`, `RecipeEngines`, `childRecipeSupported`, `unsupportedChildRecipeRefusal`) take the constant, and allows test files to spell the YAML key.
- **F6** (card 22): the `SplicePart` Stuck-with-Halt paragraph and its goto way forward are replaced by: a `SplicePart` error is returned as the step's error naming the part, its inputs being refused earlier in the same `Call` by card 12's four unknown checks. The `TestRaddleFile` row "a `SplicePart` error halts naming the goto" is removed.
- **F7** (card 22): step 3 says each part's `RaddleEntry` carries its own proposal `type`, the project's the project `type`, and both the `Entry` seam's module labels, never `sketch`; the `tasks` row asserts each part's labels in the `RaddleFiling` the recording `File` seam receives.
- **F8** (card 21): the stencil rule reads "design nothing, and put no question to the operator, before the proposal passes D6's checks", with the wording constraint against `askOperatorBounds` stated and the operator contact left to `{{.mode_rules}}`.

## Self-check

- fix-claims, LOW: card 19's allowlist comment said raddleparser is "told its paths", but its functions take bytes and values. Fixed as proposed: the phrase was dropped.
- fix-proposal, MEDIUM, and fix-affected-set, LOW (merged; fix-consistency noted the same gap as out of scope): card 25 left the hand-built `pawlengine.Paths` fillers and the shedrecipe field lists without `HandOffMarker`. Fixed with my own change.
  Edit gains `internal/shedrecipe#requirePawl`, `#zeroEnvField` and `#simpleEntryCases`.
  The Intent says `beamUpEntry` requires `HandOff`, `Notify` and `Paths.HandOffMarker`, `requirePawl`'s map and `zeroEnvField` gain them, and `validPawlDeps` and `envkit.FullEnv` set the marker and fill `HandOff`.
  I also moved the BeamUp row's unread field from `Pawl.Notify` to `Pawl.Down`, because BeamUp now reads `Notify` (`internal/shedrecipe/entries_simple_test.go:218`).
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.
