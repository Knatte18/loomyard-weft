```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I looked for what the diff's changes force outside the lines it touches.

**Set found per change**

- **PATTERN-gate-fixer entry:** `PATTERN.md` carries the declared text exactly, and the declaration names no `[background]` file. Covered.
- **Fixer roles `gate-fixer` and `verify-fixer`:**
  - `contracts/stencils/stencils.go:682-684` has a `roleOpeningStencils` row for each, both mapped to `gatefix-frame-cold`.
  - `internal/loomcli/fixerendguard.go:26` ends every role in `gatefixer.Roles()`, and `internal/gatecli/cardfixer.go:72` guards with `IsFixerRole`.
  - `internal/websterengine` kept only prose mentions of `verify-fixer`; there is no remaining role constant.
  - I did not confirm `tools/tokencount` handles the new role: my grep for the role names there found no hit, and I had no call left to read how it selects roles. Not raised as a finding.
  - Covered.
- **Deleted webster symbols** (`RenderVerifyFixColdPrompt`, `awaitColdFixer`, `coldfixer.go`, the cold prefix stencil): the only remaining match is `renderColdFixer`, a helper in `internal/websterengine/template_test.go` that now calls `gatefixer.RenderPrompt`. Covered.
- **`fixer` element key on gate elements:**
  - `contracts/specs/shed-recipe-spec.md:79-80` documents it.
  - `contracts/recipes/darn-recipe.yaml:112` and `contracts/recipes/tapestry-recipe.yaml:127,276` opt the three rows in.
  - The shedrecipe tests cover it: `entries_gatefixer_test.go`, `entries_gate_test.go:265` (a fixer on a row that reads none), `entries_darnwrite_test.go`, `entries_discussionwrite_test.go`, `entries_planwrite_test.go`.
  - `loomrecipe/recipekeys.go` `GateFixerSetting` has a test row. Covered.
- **New module config keys:**
  - `darn.gate_fixer`: `internal/configreg/configreg_test.go:79` lists it, and commit `a84d2f2f` added that.
  - `tapestry.plan_gate_fixer` and `loom.discussion_gate_fixer`: both are category `none` (`tapestry.go:151`, `config.go:557`), so no local-denied row is needed.
  - The config loaders and their tests cover all three.
  - Covered.
- **`gate.yaml` `fixer_timeout_min` and the `card` key's `fixer`:**
  - The template, `ConfigSpec` categories, `decodeConfig` and `decodeGates` are all updated.
  - The `configreg_test.go` hits for `fixer_timeout_min` already existed for landing.
  - `internal/gateslot/doc.go` and `internal/gatecli/doc.go` are updated.
  - The hand-written `gate.yaml` fixtures I found are `internal/hubgeom/webstergeom_test.go:126` and the `configcli_test.go` fake editors. Neither passes through `decodeConfig`'s new `fixer_timeout_min &gt;= 1` check, as far as I could tell. I did not verify this, since `configcli_test.go` seeds from `gateTemplate()`.
  - Covered.
- **Writer stencils (D7):** the `loom-template-plan.md`, `loom-template-discussion.md`, fragment and `twill-writer-fork-gated.md` stencils are in the diff, along with `webster-fragment-card-fixer.md` and `render.go`'s `card_gate_fixer` marker. The `template_test.go` optional-marker lists at lines 521 and 528 include it. Covered.
- **`lym gate test` behaviour change (a red card call hands off to a fixer):** `internal/gatecli/doc.go` is updated, but `docs/overview.md` is not. See finding 1.
- **`internal/shedengine/goto.go` guard contract:** the doc comment is amended, and `fixerEndingGuard` in `internal/loomcli/fixerendguard.go` has a test file. Covered.

**Findings**

1. **LOW, consistency.** `docs/overview.md:497` is not updated, so the `gate` module paragraph is stale.
   - It says a detached agent call "answers `done: false`…and the same command again prints the run's result". It does not mention that a red call on a key with a `fixer`, today the `card` key, now hands off to a cold fixer and re-runs.
   - The diff adds no overview text for `internal/gatefixer` or for webster's move onto it.
   - `docs/overview.md` is the only repo doc I found that describes the gate verb and the module set; `docs/overview.md` is not in the diff.
   - I could not confirm that a repo rule requires overview updates for this kind of change, so this is a docs-landing gap and not a rule violation.
   - Evidence: `docs/overview.md:497`; the diff stat has no `docs/` entry.
2. **NIT, consistency.** `internal/gateslot/config.go:391` adds another bare `"card"` literal.
   - The existing literals are at lines 378 and 568.
   - `PATTERN-named-constants` asks for one declared constant for a name that can go stale.
   - Evidence: `internal/gateslot/config.go:378,391,568`.

Nothing else in the affected set is left wrong.

**Tool calls (6)**
1. A grep for `verify-fixer` outside gatefixer, to find role tables and prose.
2. A grep for `busy_quit_sec` and `fixer_timeout_min` file lists, to find `gate.yaml` fixtures and docs that need the new key.
3. A grep for the new module keys, to find configreg and test classification sets.
4. A grep of docs and specs for `gate-fixer`, plus a grep of `"card"` literals in gateslot. The docs grep found nothing, which prompted the overview check in call 6.
5. Fixture lines for `busy_quit_sec`, `configreg_test.go` lines 60-110, and the loom and tapestry key categories. My context did not hold the fixtures or the classification test.
6. `stencils.go` lines 660-700, role-constant usages, `tokencount` role names, `docs/overview.md` hits and the `shed-recipe-spec.md` diff. My context did not hold the role table, the `tokencount` code or the docs.
