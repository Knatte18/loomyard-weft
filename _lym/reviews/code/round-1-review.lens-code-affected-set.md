```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I traced what reads, asserts or describes it, and checked the rule documents, stencils, PATTERN declarations and commit messages.

**Per change, the set found and whether it is covered**

1. **`template.yaml`: jacquard blocks in `speed`, `light`, `solid` and `wide`, plus `defaults.jacquard: solid`.**
   - Tests that assert the defaults or profile contents: `internal/boardcli/verbs_test.go:1077` and `internal/runprofile/runprofile_test.go` (`TestLoad`, `TestTemplateLadder`) are updated. Covered.
   - `internal/hubreconcile/ensure_integration_test.go:879-925` and `internal/configcli/configcli_integration_test.go:393` build hub files from `runprofile.ConfigTemplate()`. Their jacquard blocks already equal the derived ones, so the new reinterpretation rewrites nothing and mails no line. The mail assertions there check containment only (`strings.Contains`), so nothing goes stale. Covered.
   - The template's own header comment gains its jacquard line. Covered.
   - `docs/overview.md:404,506,534` and `contracts/specs/raddle-proposal-spec.md:44,101` describe profiles generically and list no recipe blocks. Nothing goes stale.
   - `internal/runprofile/runprofile.go` (`Settings`, `Default`, `decode`) does not enumerate recipes. Nothing goes stale.
2. **`internal/configreg/configreg.go`: the profiles module registers `profiles-jacquard-blocks` after the arras entry.**
   - Readers: `hubreconcile.applyReinterpretations` (`internal/hubreconcile/reinterpret.go:52`) and the snapshot of markers (`internal/hubreconcile/walk.go:576`), which are generic over the list.
   - The order is pinned by the new `TestLookup_ProfilesCarriesTheJacquardBlocksReinterpretationAfterTheArrasOne`.
   - The package docs (`internal/configreg/doc.go`, `internal/hubreconcile/doc.go`) describe reinterpretations generically.
   - The marker tests in `internal/hubreconcile/reinterpret_integration_test.go` use a fake ID.
   - Covered.
3. **`internal/runprofile/reinterpret.go`: new `ReinterpretJacquardBlocks` and `JacquardBlocksReinterpretationID`; the arras encoder moves into `encodeDocument`.**
   - The only caller is configreg. The arras behaviour is unchanged: the same encoder and the same line text.
   - `internal/runprofile/doc.go` gains a paragraph on the change. Covered.
   - The comments name profiles keys without their module: not covered, finding F1.
4. **`internal/runprofile/doc.go` prose.** The only finding is the bare key names in F1.
5. **Rule documents.**
   - `PATTERN.md` and `pattern/`: no entry names profile recipe blocks or this reinterpretation. `PATTERN-config-layers`' reinterpretation rule is met: the change ships as a `configreg.Reinterpretation`, applied by `hubreconcile`.
   - Declared PATTERN changes: the directive declares none, so there is nothing to check.
   - Stencil wording: the diff changes no stencil.
6. **Commit messages** (`2ef778494`, `3d4cf8bcb`).
   - Neither carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed.
   - No test check is widened. The changed `wantDefaults` and `solo` rows add the new recipe.
   - Neither message claims agreement with a reference or an improvement.

**Findings**

- **F1.** Severity BLOCKING. Class `consistency`.
  - **Location:** `internal/runprofile/reinterpret.go:81-91` and `:148-155` (the doc comments of `ReinterpretJacquardBlocks` and `reinterpretJacquardDefault`), and `internal/runprofile/doc.go:31-32`.
  - **Summary:** these comments name profiles config keys bare: `defaults.jacquard` and `defaults.tapestry`. `PATTERN-config-key-names` asks for `<module>.<dotted path>`, so these should read `profiles.defaults.jacquard` and `profiles.defaults.tapestry`.
  - **Evidence:** the diff's own messages render these exact keys in full through `configengine.KeyName(ModuleName, DefaultsEntry+"."+jacquardRecipe)` (`reinterpret.go:159`, `:162`, `:182`), so the diff itself treats them as config keys.
  - **Mitigating:** the existing `defaults.arras` mentions (`doc.go:24`, `reinterpret.go:37`) are bare too, and they survived the full-name sweep in `e2ac87a8a`. That commit's message says the guard skips seed-only modules and that "nothing keeps comments in full form mechanically". The handler may judge this a reading the repository accepts for profiles entries. The rule's own text still covers comments.

**Tool calls**
1. Grep for tests and docs that read the template defaults: I needed every consumer of the template shape outside the diff.
2. Grep of `docs/overview.md`, the raddle spec and the profile tests in configcli and hubreconcile: I needed their exact mentions of profiles.
3. `ensure_integration_test.go:900-935` and a grep for marker tests: I needed to know whether the walk tests assert the mailed lines and markers exactly.
4. Grep of PATTERN, `pattern/`, `docs` and `contracts`, plus the template's header: I needed any rule document naming the recipe blocks or the arras reinterpretation.
5. Grep of `runprofile.go` for recipe lists: I needed to rule out an enumeration of recipes that the diff would leave stale.
