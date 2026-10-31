---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomrecipe/loomrecipe.go:18
    summary: "TapestryRetiredRows has no Loom-Preflight to Tapestry-Preflight entry, which the decision record Constraints require, so a run parked on Loom-Preflight cannot resume"
    fix: "In internal/loomrecipe/loomrecipe.go add the entry `Loom-Preflight` mapped to `loomshed.NameTapestryPreflight` to the map TapestryRetiredRows returns. Add a row for it to TestRenameRetiredRows in internal/loomcli/wiring_darn_test.go (a tapestry run parked on Loom-Preflight resumes on loomshed.NameTapestryPreflight). Update the doc comments of TapestryRetiredRows, internal/loomrecipe/doc.go:27 and the retired-row paragraph of contracts/recipes/tapestry-recipe.yaml:58 to name the new entry. This is the one retired-row entry the parent's withdrawal leaves in place; it adds no alias for the recipe name."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shedrun/seed_test.go:160
    summary: "D6 is pinned by no test, because nothing asserts that the old recipe name loom is refused as unknown and that the refusal lists tapestry"
    fix: "Add a refusal row for the value loom to TestValidateRecipe in internal/shedrun/seed_test.go, asserting the error text contains unknown recipe and lists tapestry. Add the same assertion for a shed seed or arm call given --recipe loom in internal/shedcli/cli_test.go next to the existing unknown recipe rows."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: README.md:58
    summary: "README.md and docs/overview.md still name loom as the default recipe and as the meaning of an empty recipe, which D4 changed to tapestry"
    fix: "In README.md:58 write tapestry for the default recipe. In docs/overview.md:650 write tapestry in the recipe list and in the empty meaning phrase. Keep every loom that names the general run."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: tools/sandbox/SANDBOX-CORE-SUITE.md:250
    summary: "The sandbox suite tells the operator to write a seed with recipe loom, spaced form, which the tables missed; the seed is now refused as an unknown recipe"
    fix: "In tools/sandbox/SANDBOX-CORE-SUITE.md:250 change the seed to recipe tapestry. Then grep tools/sandbox for any other spaced seed form with the old recipe value and change it the same way."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/arm.go:203
    summary: "A comment still names the retired recipe value loom for a prime seed"
    fix: "In internal/battencli/arm.go:203 write tapestry for the value a prime seed carries."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/shed-recipe-spec.md:113
    summary: "The recipe spec still says loom's real row list and loom-specific constructors for the variant's own rows, which commit b44cdabf says were changed"
    fix: "In contracts/specs/shed-recipe-spec.md lines 112 and 113 say tapestry's real row list and tapestry-specific constructors. Keep internal/loomrecipe and loomshed names."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/task_test.go:187
    summary: "Tests still use loom as a recipe name in board and run profile data and in recipe-keyed map keys, which keeps the retired name alive as variant data"
    fix: "Change the recipe value to tapestry in internal/boardengine/task_test.go:187 and board_test.go:249, in the recipe block and default keys of internal/runprofile/runprofile_test.go:56-61, internal/configreg/runsettings_test.go:475 and internal/configcli/configcli_test.go:1118, and in the recipe-keyed map keys at internal/loomrecipe/recipe_test.go:99, decisionswitches_test.go:26, fansettings_test.go:109, internal/testkit/envkit/envkit_test.go:76 and internal/loomcli/wiring_darn_test.go:240. Keep every loom that is the config module key inside a profile block."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: internal/lenses/doc.go:1
    summary: "lenses and reviewbudget docs say the fan key grammar tapestry and darn share, although the shared grammar belongs to the loom config module that darn and pawl also read, and line 4 of the same file still says loom"
    fix: "In internal/lenses/doc.go lines 1 and 13 and internal/reviewbudget/doc.go restore loom where the text means the loom config module and its keys. Read each phrase from the commit edea6b48 diff and keep tapestry only where the text means the recipe."
---

### [BLOCKING] Retired-row entry for Loom-Preflight is missing

**Location:** internal/loomrecipe/loomrecipe.go:18
**Issue:** The decision record's Constraints say row names are durable resume identities, so the `Loom-Preflight` rename goes with a retired-row entry, and the focus directive keeps it outside the parent's withdrawal. `TapestryRetiredRows` returns only `Merge-In`, so `peers.go:88` and `arm.go:197` find no successor for a status parked on the old row, and no test row pins the rename. Confirmed by reading the function body.

### [MEDIUM] No test pins D6

**Location:** internal/shedrun/seed_test.go:160
**Issue:** `TestValidateRecipe` refuses `beam` and `someday` only. Nothing asserts that `loom` is refused as unknown with `tapestry` listed, which is the acceptance criterion for D6 and the absence of an alias.

### [MEDIUM] README and overview keep loom as the default recipe

**Location:** README.md:58
**Issue:** `README.md:58` and `docs/overview.md:650` state the recipe vocabulary and the empty-recipe default as `loom`, while D4 and the board README render `tapestry`.

### [MEDIUM] Sandbox suite seed uses the retired recipe value

**Location:** tools/sandbox/SANDBOX-CORE-SUITE.md:250
**Issue:** The spaced form `{"recipe": "loom", ...}` is not a key of the value table, so it stayed. With no alias, the seed written from this line is refused.

### [LOW] Comment names the retired recipe value

**Location:** internal/battencli/arm.go:203
**Issue:** The comment says a prime seed carrying `"loom"`; the value is now `"tapestry"`.

### [LOW] Recipe spec keeps loom for the variant's rows

**Location:** contracts/specs/shed-recipe-spec.md:113
**Issue:** Lines 112 and 113 say loom's row list and loom-specific constructors for rows only the tapestry recipe runs.

### [LOW] Tests keep loom as a recipe name

**Location:** internal/boardengine/task_test.go:187
**Issue:** Recipe values and recipe-keyed map keys in several tests still say `loom`; some pass only because the board and run profile layers do not validate the vocabulary.

### [LOW] Over-renamed config-module prose in two package docs

**Location:** internal/lenses/doc.go:1
**Issue:** The shared fan key grammar is the `loom` config module's, read by darn and pawl too; D1 keeps that name. The phrase table turned "loom and darn share" into "tapestry and darn share", while line 4 of the same file still says loom's twill writers.

## Rejected

- Lens code-correctness F1 (`.git/` exclusion does not cover a worktree's `.git` file): no key matches the `gitdir:` text, so no change results; not a defect of the diff.
- Lens code-tests F3 (`moves` entries two and three not exercised): table data of a one-shot program, and a second moved file would only repeat the covered move behaviour.
- Lens code-correctness `lym ` guard split across a newline: a latent gap the lens itself marks as not observed.
- Lens code-tests items 5 and 9 (refusal text of `ProjectRecipe`, unexercised value fragments): a message wording and one-off fixture rewrites that the gate's own runs check.
