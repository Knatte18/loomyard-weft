---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/yamlengine/layers.go:193
    summary: A hub-side single gate set such as `lyx config gate --set gates.card=[0,1]` writes a sparse base holding only that gate, and the load carries the open map whole, so every other gate list is lost and refused at use
    fix: In internal/gateslot/config.go, internal/gateslot/template.yaml's registry entry and internal/configreg/configreg.go, declare `gates` a per-name map instead of an open map (add `ConfigPerNameMaps()` returning `gates` to gateslot, set `PerNameMaps` on the `gate` Module and on `ConfigSpec`, and drop `gates` from `ConfigOpenMaps`), so `Merge` descends per gate name, `SetSparse` writes one name and `DropTemplateEqualNames` drops template-equal names; keep `tiers` and `tier_parallel` open maps. Then add one row in internal/configcli/configcli_test.go that runs a hub `--set gates.landing=[0,1]` and loads with gateslot.LoadConfig, asserting `landing` is [0,1] and `card` still resolves to the template list.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/hubreconcile/ensure_integration_test.go:172
    summary: The parent's darn fixture rewrites the text verify with an empty value, which after the task retired darn verify matches only inside tree_scope_verify, so it plants an invalid value and passes by substring accident, and no test sends a retired key through the walk strip
    fix: >-
      In preMigrationBase's darn case, plant a valid tuned darn unit instead (for example set verify_attempts to 5 with replaceOnce on the template's verify_attempts line),
      and change assertBoardStrippedOnce and its doc comment to expect that unit and to drop the hub's own verify command wording.
      Add a row to the same walk test that seeds the hub darn.yaml with the retired verify key set to make verify, the gate.yaml with go_parallel set to 8 and the landing.yaml with publish_verify set to x, each beside a template-equal unit,
      and asserts after the walk that the retired keys and the equal units are gone, a tuned unit is kept, and the marker is written.
---

### [MEDIUM] Single gate set in a hub drops the other gate lists

**Location:** internal/yamlengine/layers.go:193, internal/yamlengine/layers.go:72-86, internal/gateslot/tiers.go:230, internal/gateslot/config.go:59
**Issue:** The parent made every hub base write sparse, so `SetSparse` writes only the entry set into an empty `gates` mapping. The task declared `gates` an open map whose template value holds the `card` to `publish` lists, and `Merge` treats an open-map key as one whole unit, so the base's `gates` replaces the template's. After one `--set gates.<name>=...`, `TierList` refuses every other gate with `gates holds no list for`. Before the merge `SetValues` scaffolded from the template and kept the defaults. The decision record's D12 sets every gate, so it works only when all are set; a later template change to the lists also never reaches a hub that pinned the whole map. No test sets a `gates.*` entry through the hub path.

### [MEDIUM] Parent's darn fixture passes by accident and no retired key goes through the walk strip

**Location:** internal/hubreconcile/ensure_integration_test.go:172, internal/hubreconcile/ensure_integration_test.go:610
**Issue:** `replaceOnce(..., "verify: \"\"", "verify: make verify")` was written for darn's `verify` key. The task retired it, so the text now matches inside `tree_scope_verify: ""` and the fixture seeds `tree_scope_verify: make verify`, a value the task's darn load refuses. `assertBoardStrippedOnce` passes only because `verify: make verify` is a substring of that line; its comment is stale. `lyx gate test --tags integration ./internal/hubreconcile` passed on the merged tree, so nothing catches it. Prune runs before the strip in both `ReconcileHubWideAt` and `reconcileSparseFile`, so the two passes agree on a file holding a retired key and a template-equal unit, but no test pins that.

## Rejected

- Stale `verify` wording in contracts/recipes/darn-recipe.yaml:157, docs/overview.md:269 and :479, and `--tags` in contracts/specs/loom-plan-spec.md:525: these lines live on the task side alone, so they are not the interaction (the Webster review's subject).
- Former-values table in configsync against the merged darn template: every darn entry names a key the merged template still holds.
- Landing, Merge-In, envkit and lyxbin composition: both sides' seams merged without overlap and compile.
- PATTERN.md length and declared entries: all lines are under the raised limit and the declared texts sit in the background files.
