```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1–F8), building a case where it is wrong or breaks something elsewhere.

**[MEDIUM] Card 25, rewritten sentence "`beamUpEntry` requires the new seam and path; `pawlDeps` fills the `HandOff` seam, and the path reaches it through … `hubgeom.PawlPaths(location)`"**

- **Summary:** The F2 proposal covers only the production filler of `pawlengine.Paths`. Several test fixtures and one field list build the struct by hand, and no card targets them. So the new required path breaks those fixtures, and the existing field-list test misses the new field.
- **Evidence:**
  - `internal/testkit/envkit/envkit.go:381` builds `Paths{… AwaitMarker: filepath.Join(dir, "await-entered")}` literally. Card 25 lists `envkit#FullEnv` under Edit, but its Intent changes FullEnv only to make its `Notify` fake report posted. It never says FullEnv fills `HandOffMarker`.
  - `internal/shedrecipe/fixture_test.go:70` builds the same literal, and no card targets it. If `beamUpEntry` requires `HandOffMarker`, an entry built from this fixture is refused.
  - `internal/shedrecipe/entries_pawl.go:13` defines `pawlPathFields` (`"Paths.LandRequest", "Paths.DobbyStopMarker", "Paths.AwaitMarker"`) and line 23 maps them. These are what `beamUpEntry` "requires", but the card never says the list gains `Paths.HandOffMarker`.
  - `internal/shedrecipe/entries_simple_test.go:100` and `:217`/`:225` pin the same field set (the unset-field case and `validatedFields`). Neither is a target of card 25.
- **Fix:** add those sites as targets. Say that `pawlPathFields`, the hand-built `Paths` in `envkit.FullEnv` and in shedrecipe's fixture, and the `entries_simple_test.go` field lists gain `HandOffMarker`. Alternatively, drop the claim that `beamUpEntry` requires the path.

Proposals I tried to break and could not:
- **F1:** boardengine's dependencies hold no loomshed import, and the allowlist governs direct imports only.
- **F3:** `return_note` is not a reserved marker, so `Optional` accepts it (`internal/seatengine/table.go`, the check that an `Optional` name is not a reserved marker).
- **F4:** comparing keys from a raw map passes the `DisallowUnknownFields` scan.
- **F5:** a constant as the map key is sound.
- **F6:** `Check(filing=true)` rejects every input that reaches `SplicePart`.
- **F7:** `validateLabels` (`internal/boardengine/labels.go:14`) needs only one type label, and accepts module labels that are in the vocabulary.
- **F8:** the example sentence holds "before" and none of the phrases in `askOperatorPhrases`.

Tool calls:
1. `sed` of `internal/seatengine/table.go:215-240` and `internal/boardengine/labels.go` `validateLabels`: needed to test the F3 and F7 proposals against the `Optional` and label rules, which my context lacked.
2. `grep` for `AwaitMarker` and `PawlPaths` in envkit and shedrecipe: needed to test whether the F2 proposal covers every builder of `pawlengine.Paths`, which my context lacked.
