```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens over the amended cards 07, 08, 23 and 25. For each card I checked test seams and tier tags, doc text the card makes false, and card order.

**Per-card list**

- **Card 07, report-config**
  - Test seams: none added. Held.
  - Tier tags: `config_test.go` calls `tiergate.Require(t, 0)` at line 18, and `testmain_test.go` holds only `TestMain`, which takes no tier. Held.
  - Doc text: `internal/configreg/doc.go` does not list the config modules. `docs/overview.md:319` already names `internal/reportconfig`. Held.
  - Card order: needs nothing from a later card. Held.
- **Card 08, report-cost**
  - Test seams: none added. Held.
  - Tier tags: `cost_test.go:110` calls `tiergate.Require(t, 0)`. Held.
  - Doc text: the card edits `reportengine/doc.go`, `reportcli/doc.go` and `docs/overview.md`. Held.
  - Card order: it uses `plan:internal/reportconfig#LoadConfig` from card 7, which comes earlier. Held.
- **Card 23, raddle-recipe**
  - Test seams: **broken.** The card adds two fields to `shedrecipe.Env`, `RaddleWrite` and `RaddleFile`. It does not edit `envkit.FullEnv`, and it does not edit the `//lym:seam` markers of the envkit reflective tests.
  - Tier tags: every new test is named tier0. Held.
  - Doc text: the card lists the recipe-list doc files. Held.
  - Card order: **broken**, as a result of the seam gap. The card's own gate fails, because the only card that edits `FullEnv` is card 25, which comes later and fills only the pawl seam.
- **Card 25, pawl-raddle-handoff**
  - Test seams: the card adds `Deps.HandOff` and `Paths.HandOffMarker`, and it edits `envkit#FullEnv`, `validPawlDeps` and `world.deps`. Held.
  - Doc text: the card edits `pawlengine/doc.go`, and `internal/hubgeom/doc.go` does not list the pawl paths (a grep found none). Held.
  - Card order: it uses `RaddleStore.Holds`, which an earlier card creates. Held.

**Findings**

1. **BLOCKING, design, `_lym/plan/23-raddle-recipe.md` lines 34, 82–86 and 98.**
   - **Summary:** card 23 adds `Env.RaddleWrite` (`RaddleWriteEnv`) and `Env.RaddleFile` (`loomshed.RaddleFileDeps`) but does not fill them in `envkit.FullEnv`. It also adds no `//lym:seam` markers for the new seam structs. The envkit tests appear only under `**Uses:**`.
   - **Effect:** `TestFullEnv_FillsEverySeam` would fail at card 23's own gate, and so would `TestSeamMarkers_ReflectiveTestsAreMarked`, which checks that a test reaching `NilSeams` marks every struct it reflects over. No earlier card fills these fields; card 25, which is later, fills only the pawl seam.
   - **Evidence:**
     - `internal/testkit/envkit/envkit.go:299` declares `FullEnv`, and line 487 fills `RecordWrite` field by field. Line 632's `walkNil` reflects over the whole `Env`.
     - `internal/testkit/envkit/envkit_test.go:40`, `:110` and `:153` carry `//lym:seam internal/loomshed#RecordWriteDeps` on the envkit tests, and lines 281–282 hold the "markers miss" refusal.
     - `_lym/plan/25-pawl-raddle-handoff.md:17` is the only card that edits `envkit#FullEnv`.
   - **Fix:** in card 23, move these from `**Uses:**` to `**Edit:**`:
     - `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
     - `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
     - `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

     Also add `internal/testkit/envkit#FullEnv` under `**Edit:**`. The Intent should say that `FullEnv` fills both raddle fields and that those tests' markers name `shedrecipe#RaddleWriteEnv`, `loomshed#RaddleWriteDeps` and `loomshed#RaddleFileDeps`. The same holds for the `TestWire_EverySeamFilled` tests in `internal/loomcli` and `internal/battencli`, if their markers enumerate the seam structs; I did not check those.

Tool calls:
1. Read the head and tier calls of `reportconfig` and `cost_test.go`, the `configreg` doc and the `docs/overview.md` module lines. My context did not show the new test files' tier tags or which docs list the config modules.
2. Grepped `envkit` for `FullEnv` and the seam-marker tests. My context did not show how the envkit seam tests cover `Env`.
3. Grepped `hubgeom/doc.go` for a list of pawl paths. My context lacked that doc's content, which card 25 might make false.
4. Grepped the plan cards for envkit edits and checked the `tiergate` calls. I needed to know whether any earlier card fills the new `Env` fields.
