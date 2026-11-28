```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I traced what each change forces outside the lines it touches.

**Per change: the affected set, covered or not**

- **Recipe vocabulary gains `jacquard`** (`RecipeNames`, `OfferedRecipeNames`, `ValidateRecipe`):
  - covered: `shedrun/seed_test.go`, `shedcli/table.go`, `shedcli/table_test.go`, the `loomrecipe` lists and `embeddedRecipes`, and `docs/overview.md`.
  - not covered: `battencli/wire.go:611/617` (batten's supported child recipes), `reportengine/runs.go:26` and `reportengine/timingspans.go:31`. The amended entry leaves all three to later parts, so none of them is a finding.
- **tapestry becomes two block rows with entry `Motif`, and its rows move to `jacquard-recipe.yaml`:**
  - covered: the moved rows and top keys are byte-identical to the base (checked by diff). The motif header, the recipes.go doc, `overview.md` and the loomrecipe tests (blocks, progress, entry numbers, specfill, shape, the sequence tests and `setRowProducer`) are all updated. So are the battencli stage numbers and the envkit Segments test.
  - not covered: `contracts/recipes/arras-recipe.yaml:15`, the `internal/loomshed/loomshed.go:5,12` comments, `internal/loomshed/doc.go:3`, `contracts/specs/shed-recipe-spec.md:12`, and the jacquard comment at line 263 (`jacquard-recipe.yaml:263`). Findings F1–F3 below.
- **`Expand` run-ending rule (D3):**
  - covered: `expand.go` doc comment, `expand_test.go`.
  - not covered: `contracts/specs/shed-recipe-spec.md:208`. Finding F4.
- **Loom contract widened to record and seed rows (D4):**
  - covered: `loomcontract.go` doc comment, `build_test.go`, `shedrecipe/doc.go`, `registry.go`.
  - not covered: `contracts/specs/shed-recipe-spec.md:224,228,233-235`. Finding F4.
- **`RecordEngine` returns `RecordSeed`:**
  - covered: `recipekeys_test.go` and the callers `wiring.go:434/841`. `validate.go:78` and `entries_gate.go:211` compare against `DiscussionWrite` and so behave correctly for jacquard. `decision.go:1729` (`rowWritten`) gives false for jacquard; the decision-verb work is left to later parts by the amended entry, so no finding.
- **New engine `RecordSeed`, `Env.RecordSeed`, `RecordSeedDeps`:**
  - covered: registry, conformance and Names tests; the `//lym:seam` lines and `intentionallyNil` in battencli and loomcli; envkit `FullEnv` and its test; the shedrecipe test fixture; the interrupt policies.
- **New row names `Record-Seed` and `Jacquard`:**
  - covered: `loomshed.go` constants, `interruptpolicy.go`, `shape_test` table.
- **Seed `parent` param read in the new code:** checked under "The repository's own rules". Finding F5.
- **Declared PATTERN changes:** the directive declares none, and the diff touches neither `PATTERN.md` nor `pattern/`. Nothing to raise.
- **Commit messages and departures:**
  - There are no `Pattern-Departure:` trailers.
  - The one `Loosened-Check:` trailer is on `2ff32d728`, for `TestWire_EverySeamFilled`. It covers new intentionally-nil seams, not a widened existing assertion. It also names a measured cause, the test reporting those three seams nil. Nothing to raise.

**Findings**

- **F1 · MEDIUM · consistency · `contracts/recipes/arras-recipe.yaml:15`**
  The arras header still says its rows "follow the shared-segment ring tapestry-recipe.yaml's header states". The tapestry header no longer states that ring; the rule moved to `jacquard-recipe.yaml`'s header (diff 002, which removed it from tapestry).
  Evidence: `arras-recipe.yaml:15`; `tapestry-recipe.yaml` new header lines 1–19.
  Fix: point it at `jacquard-recipe.yaml`'s header.

- **F2 · LOW · consistency · `internal/loomshed/loomshed.go:5,12` and `internal/loomshed/doc.go:3`**
  These comments still describe tapestry's file as holding the rows:
  - "The producer names, verbatim per the rows of contracts/recipes/tapestry-recipe.yaml"
  - "tapestry-recipe.yaml spells the same names as YAML strings"
  - "Loom's ordered producer list lives in contracts/recipes/tapestry-recipe.yaml"

  The rows now live in `jacquard-recipe.yaml` and `motif-recipe.yaml`; tapestry holds only the two block rows.
  Evidence: `loomshed.go:5,12`; `doc.go:3`.

- **F3 · LOW · consistency · `contracts/recipes/jacquard-recipe.yaml:263`**
  The moved comment "fix-scope: overlay, matching the Discussion-Burler-Review row above" now names a row that is not in the jacquard file. The rows were moved verbatim, and this one comment became false.
  Evidence: `jacquard-recipe.yaml:263`.

- **F4 · MEDIUM · consistency · `contracts/specs/shed-recipe-spec.md:208,224-235` (and line 12)**
  The shed recipe spec still states the old rules:
  - line 208: "an outermost block row must set `on_done`", which D3 now relaxes for a last always-gated block row;
  - clause 1 at line 224: "exactly one record row, an engine of `shedrecipe.LoomRecordEngines`";
  - clause 3 and the record-only form: "clause 1 is its whole contract".

  The diff changes `Expand` and `checkLoomContract`, but this spec is not updated: it does not mention the seed row, `LoomSeedEngine`, the seed-after-record order, or the seed row standing for the record row.
  Line 12 also still names `tapestry-recipe.yaml` as the data file holding each row.
  Evidence: `shed-recipe-spec.md:208,224,228,233-235,12`; `expand.go:26-36`; `loomcontract.go:16-19`.

- **F5 · BLOCKING · design (PATTERN-code-conventions, Named constants) · `internal/loomcli/wiring.go:569`, `internal/loomshed/recordseed.go:47`**
  The seed's `parent` param name is now spelled in two new places:
  - `wiring.go:569` reads it as the bare literal `seed.Params["parent"]`;
  - `recordseed.go:47` declares its own private constant `seedParentParam = "parent"`.

  The existing literal at `sharedbootstrap.go:293` (`Params: map[string]string{"parent": parent}`) stays as it was. The rule asks for one named constant, declared once and referenced everywhere, never a repeated literal; the diff adds a literal and a second definition.
  Fix: declare one exported constant for the loom seed's parent param and use it at all three sites.
  Evidence: `wiring.go:569`; `recordseed.go:47`; `sharedbootstrap.go:293`.

**Tool calls**

1. Grep for files naming `tapestry-recipe.yaml`, and for Preflight-as-entry prose, to find docs, spec and fixture readers the manifest lacked.
2. Grep of `loomshed.go`, `loomshed/doc.go`, `shed-recipe-spec.md` and `arras-recipe.yaml` for their tapestry references and the spec's block and contract rules, to place the hits from call 1.
3. Read `shed-recipe-spec.md:196-240`, `arras-recipe.yaml:10-20` and `loomshed/doc.go:1-8`, plus a diffstat of specs and pattern, to confirm the stale text and that the diff leaves it untouched.
4. Grep for other hand-written recipe-name lists, to find any vocabulary list left without jacquard.
