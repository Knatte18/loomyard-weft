```yaml
lens: code-correctness
outcome: reported
```

Task: check the correctness of the code the diff changes, one concrete case per function or branch.

**List (function or branch, case run, result)**

- `template.yaml`, `active: "wide"` plus a `wide` profile at `budget: 300000`: parsing the template and resolving gives the cost batcher named `wide`, with the `cautious` weights. Correct.
- `template.yaml`, `cautious` and `identity`: `cautious` keeps `budget: 200000` and the same weights as `wide`. `identity` is unchanged. Correct.
- `batcher.Active` with the base layer absent: resolves the embedded template, so it gets `wide`. Correct.
- `batcher.Active` with `active: ""`: resolves identity, unchanged. Correct.
- `templateOrientation` in `config.go:246-256`, which still reads the `cautious` profile's `weights.orientation`: the `cautious` profile still exists, so it is non-nil and returns 31400. Correct, but it now depends on `cautious` staying in the template.
- `configsync.formerTemplateValues["batcher"]["active"] = "\"cautious\""` (`configsync.go:52`):
  - Present `active: "cautious"`: it differs from the new template's `"wide"`, so the template-equal strip leaves it. `stripFormerValues` then compares the leaf and removes it, so the hub inherits `wide`. Correct.
  - `active: "wide"`, `""`, `identity` or a custom name: not equal to `"cautious"`, so it is kept. Correct.
  - Absent `active`: `Strip` has no unit to compare, so nothing happens. Correct.
  - Non-string value (`active: 5`, or a mapping or sequence): it can't equal the string scalar, so it is kept. Correct.
  - Empty file: `stripFormerValues` returns early on `len(bytes.TrimSpace(current)) == 0`. Correct.
- The same strip, `active: "cautious"` chosen on purpose: a hub that picked `cautious` by choice is stripped and moves to `wide`. The file can't tell that case from an unmigrated one, and `configsync.go:49-52` doesn't mention it. The commit message says the strip moves unmigrated hubs; it does not name deliberate choosers. Unavoidable given the data. Finding F1 below.
- `profiles` former value `cautious:\n  budget: 450000\n`: unchanged. Strip works per leaf, so a hub's `profiles.cautious.budget: 450000` is still stripped. Correct.
- `formerValuesDoc("batcher")` with two entries: names are sorted (`active`, `profiles`), the `"cautious"` scalar parses to a node, and the doc marshals. Correct.
- `config_test.go` `missingThresholdErrors`: the replacer renames `  wide:` to `  mine:`, and `"    budget: "` also matches the wide block. It errors with `mine` and `budget`. `cautious` still exists with a valid budget, but `active` is `mine`. Correct.
- `config_test.go` `retiredAloneAboveErrors`: `strings.Replace(template, "    budget: ", ..., 1)` hits the first `budget`, which is in `wide`, and the error names `wide`. Correct.
- `config_test.go` `nonPositiveBudgetErrors`: replaces `budget: 300000` with `budget: 0`, which appears only in `wide`. Correct.
- `config_test.go` `retiredMasterBaseErrors`: replaces `orientation: 31400` with `master_base: 52000`, hitting the first occurrence, which is in `wide`, so the error names `wide`. Correct.
- `config_test.go` `maxCardsErrors`: replaces `max_cards: 6` with `max_cards: 1`, hitting `wide` first. Correct.
- `configsync_test.go` `TestReconcileAll` `masterBaseRewrites`: the same first-occurrence replace hits `wide`. Correct.

**Findings**

- **F1, LOW, design.** `internal/configsync/configsync.go:52`. The strip of `active: "cautious"` also moves a hub that chose `cautious` on purpose, because the value can't be told from the old template default. The Go comment on `formerTemplateValues` (lines 40-42) doesn't say so. The commit message says the strip moves unmigrated hubs, not that it also moves deliberate choosers. Fix: say so in the comment, or accept it as unavoidable. This is a documentation gap, not wrong behavior.

No other defects found in this lens.

**Tool calls**
1. `grep` for `formerValuesDoc`, `formerTemplateValues` and the strip helpers: to find where the former values are applied, which the manifest lacked.
2. `sed -n 655,720p internal/configsync/configsync.go`: to read `stripFormerValues` and `stripBase` and confirm the leaf-comparison semantics.
