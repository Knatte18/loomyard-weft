# Card 3 — yamlengine: a set to or over a null leaf takes its text's tag

**Edit:**
- `internal/yamlengine#SetValues`
- `internal/yamlengine#SetSparse`
- `internal/yamlengine#TestSetValues`
- `internal/yamlengine#TestSetSparse`

**Intent:**
`lyx config <module> --set` writes a scalar under the tag of the leaf it lands on, which goes wrong in both directions once a template holds a null leaf, as card 4's three null loom budgets do.
Null to value: `lyx config loom --set plan_max_bounces=2` over the template's `~` writes `plan_max_bounces: !!null 2`, which loads as null, so the per-perch way forward card 1's hard-below-soft refusal names, and any operator set of the null budgets, would silently inherit.
Value to null: `--set plan_max_bounces=~` over a hub default file holding `plan_max_bounces: 2` writes `plan_max_bounces: !!int ~`, since `applyExistingOverrides` has already copied the existing leaf's `!!int` tag onto the working tree before the pairs apply, and the base layer is written without validation, so the hub's loom.yaml no longer reads as null for that key, though card 1's and card 4's refusals tell the operator to set a budget to null.

The rule keys on the text being set as well as on the leaf, never on the template's tag alone:
a scalar whose own text is a spelled null (`~`, `null`, `Null`, `NULL`), or one set on a leaf whose tag is `!!null`, takes the tag its text resolves to, null for `~` and `null`, int for `2`.
An empty value is not a spelled null and keeps its leaf's tag, so `--set <key>=` on a string leaf still writes an empty string.

- `SetValues` applies the rule where a leaf's `.Value` is assigned, testing the working-tree leaf's tag at that point, which `applyExistingOverrides` has already given the existing file's tag; the template's tag cannot be the test there.
- `SetSparse` applies it where the new node takes the template leaf's tag, for both directions.

Every other set keeps its leaf's tag as today, so a string leaf set to `123` still writes a string.
A set of a spelled null on a non-null leaf now writes a null where it wrote that leaf's tag over the null text, a null being what such a set asks for.
Each doc comment states the rule.

Tests:
`TestSetValues` gains two rows over a template whose `plan_max_bounces` is `~`: setting `plan_max_bounces=2` with no existing file asserts the written document holds the integer 2 with no `!!null` tag and decodes into an `*int` as 2;
setting `plan_max_bounces=~` over an existing `plan_max_bounces: 2` asserts the written document holds a null for the key with no `!!int` tag and decodes into an `*int` as nil.
`TestSetSparse` gains one row over the same `~` template leaf that sets `plan_max_bounces` to `2` and to `~` in turn, asserting the integer 2 with no `!!null` tag, and then a null with no explicit tag, each decoding into an `*int` as 2 and as nil.

**ImpactSummary:** yamlengine's dense and sparse set give a scalar the tag its own text resolves to whenever the text or the leaf it lands on is null, so a set to or over a null config leaf reads back as written.

**Commit:** 3: yamlengine: a set to or over a null leaf takes its text's tag
