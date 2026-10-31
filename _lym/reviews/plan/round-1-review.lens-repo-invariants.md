```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens over cards 1–12 (test seams and tier tags, doc text a card falsifies, card order).

**Per-card invariants**

- **Card 1** (config):
  - Tier tags held. `config_test.go` is tier0. `testmain_test.go` is untagged, as 133 of the 146 existing `testmain_test.go` files are.
  - configreg tables held: both `TestModules_*` tests are listed.
  - Sandbox coverage held: `cmd/lym/sandbox_coverage_test.go` covers CLI modules, and card 1 adds no CLI module.
- **Card 2** (record): held. `TestPathConstructors` is listed, `record_test.go` is tier0, and the samplerengine doc is extended.
- **Card 3** (stencil): held. The stencil-wide scans are named and `spec_test.go` is tier0. `roleOpeningStencils` and `entries` are edited.
- **Card 4** (row):
  - loomshed allowlist held. `AssertImportAllowlistNoStale` holds because the producer uses both new imports.
  - Tier tag held: `samplersession_test.go` is tier0.
  - Interrupt policy held. `TestInterruptPolicyFor` lists rows explicitly, and `NameSamplerSession` gets its policy in card 6, where the recipe is assembled.
- **Card 5** (registry):
  - Env reflective seams held. Every file carrying `//lym:seam internal/shedrecipe#Env` is listed: `wiring_beam_test.go`, `wiring_seams_test.go`, `wiring_darn_test.go`, `battencli/wire_seams_test.go` and `envkit_test.go`.
  - Card order **broken** (finding F1).
- **Card 6** (recipe):
  - `TestRecipes_KeySet` and `TestRecipeNames` held, both listed.
  - `cmd/lym/driverstencil_test.go` held. It now forbids the word "sampler" in the driver stencil; no card adds it there.
  - Doc text partly broken: `RowNames`' doc is unlisted (finding F4).
- **Card 7** (vocabulary): spec texts held. Every spec line naming "tapestry or darn" that I found is listed. The landingshed `Deps` doc is listed only for the `EscalateJudgedFirst` field (see F2).
- **Card 8** (perch-off wiring): doc text **broken** (finding F2).
- **Card 9** (wiring): held. The new test file is tier0 only, `TestWire_SamplerRecipe` carries the seam markers plus `SamplerDeps`, and the tier1 test sits in an existing tier1 file.
- **Card 10** (verb): held. The test goes in the existing `commitrecords_test.go`.
- **Card 11** (decision caller): held. The test goes in the existing `decision_test.go`.
- **Card 12** (wait list):
  - Docs naming the waiting rows held: both `battencli/wire.go` doc sites (lines 615 and 674) are edited.
  - Allowlist held: `pawlengine` is added to it.
  - The idle-timeout template comment is **broken** (finding F3).

**Findings**

- **F1 — BLOCKING, design.** Location: card 5 (`05-sampler-registry-entry`), relative to card 6.
  - Card 5 cannot pass its own gate. Once card 5 registers `SamplerSession` in `shedrecipe`'s registry, `TestCoverageGuard`'s subtest `EveryRegisteredEngineIsReachedOrAllowlisted` fails. No consumer's `RecipeEngines()` reaches the new engine until card 6 adds the sampler recipe to `loomrecipe.RecipeEngines`' explicit source list.
  - Neither card lists `coverage_guard_test.go`.
  - Fix: move the embedded recipe and the `RecipeEngines` edit into card 5, or merge cards 5 and 6. Allowlisting the engine for one card is not a clean fix: the allowlist's drift subtest would then force card 6 to remove the entry again.
  - Evidence: `internal/shedrecipe/coverage_guard_test.go:51-67`, `internal/loomrecipe/names.go:28`.
- **F2 — MEDIUM, consistency.** Location: card 8 (`08-rows-keyed-wiring`); `internal/landingshed/deps.go` fields `CodeReviewOff` and `RerouteReviewed`.
  - Card 8 makes loom's `codeBudgetZero` and `rerouteReviewed` also read the perch rows' `entry_gate`.
  - The field docs still say each seam reports "whether the Code-Perch's effective review budget is 0" and "For loom that is the Code-Review perch's effective budget being above 0". Both become false on a sampler run with `code_review` false.
  - No card lists these doc lines. Card 7 lists `landingshed#Deps`, but only for the `EscalateJudgedFirst` field.
  - Evidence: `internal/landingshed/deps.go:320-326`.
- **F3 — LOW, consistency.** Location: card 12 (`12-operator-wait-rows`); `internal/loomengine/template.yaml` key `step_idle_timeout_min`.
  - The key's comment says a step showing no activity for the window is killed by the loop. After card 12, a step at an operator-wait row is never killed.
  - The comment is unlisted, though card 12's Bound says the docs will state the exemption.
  - Evidence: `internal/loomengine/template.yaml:26`.
- **F4 — LOW, consistency.** Location: card 6 (`06-sampler-recipe`); `internal/loomrecipe#RowNames`.
  - `RowNames`' doc enumerates the tapestry, darn, batten and pawl recipes and says "Any other recipe name is refused". Once card 6 embeds sampler, that is false.
  - Card 6 edits `Rows`, whose doc has the same wording, but does not list `RowNames`.
  - Evidence: `internal/loomrecipe/names.go:55-56`.

**Tool calls**

1. Grepped the `//lym:seam internal/shedrecipe#Env` markers and read `deps.go:316-328` and the head of `sandbox_coverage_test.go`: the manifest lacked the full list of Env-reflecting tests, the landingshed seam docs, and what the sandbox coverage test covers.
2. Grepped the docs naming the quiet notice, idle window and watchdog, a wakeups test, and the stencil dir registrations: needed to find doc text card 12 or card 3 falsifies.
3. Grepped docs naming Beam-Await with PR-Review, and read the `step_idle_timeout_min` template comment: needed to find which docs list the waiting rows.
4. Grepped tests over `RecipeNames()` and `InterruptPolicies`, and counted tier tags on `testmain_test.go` files: needed for the card-order checks of cards 4 and 6 and for card 1's untagged file.
5. Read `cmd/lym/driverstencil_test.go` and `loomshed/interruptpolicy_test.go`: needed to check whether card 6's vocabulary change or card 4's constant breaks them.
6. Grepped `PATTERN.md`, `pattern/` and `contracts/specs` for "tapestry and/or darn" text: needed to find pattern and spec text the vocabulary cards leave unlisted.
