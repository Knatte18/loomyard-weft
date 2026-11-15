```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens on the diff, looking for wrong results, swallowed errors and inconsistent state in the changed code.

**List: each changed function or branch, the case I ran, and whether it is correct**

- **`template.yaml`, `speed` darn block:** I read `{code_max_bounces: 0, code_hard_max_bounces: 1, code_reviewer_model: "opus[high]"}`. The YAML flow map parses and the string is quoted. The key is `loom.code_reviewer_model`, the key `loom.*` already defines. The loomengine config test at `internal/loomengine/config_test.go:669` rejects a malformed spec such as `opus[effort`, but `opus[high]` is well formed. Correct.
- **`template.yaml`, `light`, `solid` and `wide` darn blocks:** I read each one. Each carries the same key and value, and the rounds are unchanged. Correct.
- **`template.yaml`, `docs` darn block:** I read `code_reviewer_fan: ""` followed by `code_reviewer_model: "opus[high]"`. The empty fan keeps the solo round and the model is added. The `darn:` writer and `drift:` sub-blocks are untouched. Correct.
- **Untouched blocks:** The tapestry, motif, pawl, sampler, raddle and arras blocks have no diff hunks in the template. Correct.
- **`TestTemplateLadder`, sampler check:** The old test compared the sampler block's `loom` map with darn's. That comparison would now fail, since darn carries the model and sampler does not.
  - The new check reads `samplerLoom["code_max_bounces"].(int)` and `["code_hard_max_bounces"].(int)`. It uses the same cast as the pawl check beside it.
  - It compares them with `perch(tt.darnCode)`. I ran `speed` (0 and 1) against the template line 37, which gives 0 and 1, and `solid` (2 and 3), which gives 2 and 3. Correct for every profile.
  - The old check also caught any extra or missing key in the sampler `loom` map. That now goes unchecked apart from the fan and model keys. This is a coverage loss for the tests lens, not a runtime defect.
- **`TestTemplateLadder`, `blockKeys` loop:** I added `sampler` and checked the new model assertion.
  - For a block named `darn`, the model must be present and equal `opus[high]`. For tapestry, pawl and sampler, it must be absent.
  - The fan check on the sampler block is correct only if no `tt.solo` entry names `"sampler"`. The sampler `loom` map holds no `code_reviewer_fan` in any profile, and the `speed` sampler line 37 confirms it. Correct.
  - The `speed` tapestry block keeps its fan and has no model, and the loop agrees. Correct.
- **`TestTemplateDocsProfile`:** The expected map gains `"code_reviewer_model": "opus[high]"`, which matches template line 94. Correct.
- **`runsettings_test.go` row "no profiles file takes the template's default":**
  - **Expected output:** `code_hard_max_bounces: 2\ncode_max_bounces: 1\ncode_reviewer_model: opus[high]\n`.
  - **Key order:** The sibling `configreg/runsettings_test.go:201` shows the writer emits keys alphabetically, and this expected output follows that order.
  - **`light` default:** The `light` darn block now holds the model. Correct.
- **`configsync_test.go` row `hubFile: "light:\n  darn:\n    loom: {code_max_bounces: 4}\n"`:**
  - **Reconcile:** With no base file, the reconcile adds the missing keys from the template, and `code_reviewer_model` now arrives with `code_hard_max_bounces`.
  - **Added check:** `wantAddedHas` is a subset check, so the extra added path does not fail it.
  - **Rewritten text:** `wantContains` matches the prefix `code_hard_max_bounces: 2, code_reviewer_model: `, which holds whether or not the value is quoted. The test only reads from there, so it is correct.
- **`configsync_test.go` and `hubreconcile` fixtures (`lightDarn`, `lightDarnSolo`, `lightDarnOperator`, `solidDarn*`):** The test's `strings.Replace(..., 1)` calls depend on the constants matching the template text byte for byte.
  - **Template match:** Every constant ends with `code_reviewer_model: "opus[high]"}\n`, which matches the new template lines 48 and 64 exactly. The `solid` constants match line 64 and the `light` ones match line 48.
  - **Operator-kept row:** `lightDarnOperator` still sets its own `code_reviewer_fan: standard`, and the reconcile keeps that value. Correct.
- **`configcli_integration_test.go` constants:** These use the same pattern. I checked the `light`/`solid` text against template lines 48 and 64. `lightDarnSolo` puts `code_reviewer_fan: ""` before the model, so it is a valid older-state fixture. Correct.

**Findings:** None for this lens. Every changed branch I ran gives the expected result.

**Outside my lens:** The reconcile fixtures build their "older template" by editing the new template. None of them models a real earlier profiles file that lacks `code_reviewer_model` altogether. The "an unedited earlier file picks up the new model" path therefore has no explicit fixture. The tests lens should judge that.

**Tool calls:**
1. I ran `sed` on `internal/configsync/configsync_test.go` lines 940–1000 and 1040–1120. My context lacked the table's other rows and the assertion loop, which I needed to check the `wantAddedHas` and `wantContains` semantics.
