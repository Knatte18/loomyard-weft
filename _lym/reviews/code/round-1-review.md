---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/configsync/configsync_test.go:947
    summary: no seed-only reconcile row pins the new code_reviewer_model itself, an unedited earlier file picking it up or an operator's own value kept
    fix: In TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate add one row whose base and hub file are the template with the darn block's code_reviewer_model removed, expecting wantAddedHas light.darn.loom.code_reviewer_model and wantContains the opus[high] line; add a second row whose hub file holds code_reviewer_model "sonnet[high]" over that same base, expecting wantDiffers to name the key and the hub file's value kept in wantContains.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: internal/runprofile/runprofile_test.go:215
    summary: TestTemplateDocsProfile and TestTemplateLadder are listed as contained yet pin the opus[high] values no covering test asserts, and carry no testtiming keep marker
    fix: Add a `//testtiming:keep` comment line directly above `func TestTemplateLadder` and above `func TestTemplateDocsProfile` in internal/runprofile/runprofile_test.go, with a reason naming what each pins, the template's per-block code_reviewer_model values and the docs darn block's full settings map.
---

### [LOW] No reconcile row pins the new model key

**Location:** internal/configsync/configsync_test.go:947
**Issue:** The decision record asks for the default to reach hubs through the seed-only reconcile while an operator's own value stays.
Every older-template fixture in `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` and `TestConfigOverRealHub` already holds `code_reviewer_model: "opus[high]"` and differs from the template only by the fan or the round counts.
No row shows a file without the key gaining it, or an operator's different model being kept and listed under differs.
The generic add and keep logic is pinned by the fan and sampler rows, so this is a gap in the headline claim, not in new machinery.

### [LOW] Contained tests lack the keep marker

**Location:** internal/runprofile/runprofile_test.go:215
**Issue:** The redundancy report lists both tests as contained.
`TestSettings`, `TestLoad` and `TestFormerRecipeNameRefusal` do not assert `code_reviewer_model`, so the new assertions pin values coverage blocks cannot see, and the tests are accepted rather than folded.
Neither test carries the `//testtiming:keep <reason>` marker the report's rule asks for.

## Rejected

- Sampler block should also take the Opus reviewer (code-correctness, focus item 4): the entry's Expected text names only darn's Code-Review reviewer, so keeping the hub's model on sampler is no defect.
- Old whole-map sampler-equals-darn check dropped (code-tests F2): the new check pins the rounds the sampler must carry, the fan check covers the one other key a block holds, and the template carries no stray key.
- Header and `doc.go` word the model differently, "an Opus model-spec" against `opus[high]` (code-affected-set): both agree with the template as left.
- Commit-message claims about tapestry, pawl and sampler (focus item 6): they match the template, and no `Pattern-Departure:` trailer is present or needed.
