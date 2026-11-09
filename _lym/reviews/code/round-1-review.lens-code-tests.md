```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it.

**Per behaviour: the test that pins it**

1. **Each case loads to the template's own load plus its named fields.**
   - `TestLoadConfig_Loads` in `internal/loomengine/config_test.go:448-480` pins this. `reflect.DeepEqual(got, want)` against `templateLoad` plus `mutate` fails if any other field moves.
   - Cases with a no-op `mutate` (the retired `selfreport` and `fix_start` keys, `config_test.go:205-209` and `297-301`) still assert that nothing else changed.
   - The `wantSoft` cases (`config_test.go:408-418`) keep their `Caps(perch).Soft` assertions (`config_test.go:474-478`).

2. **The hub-stack subtests use the same base.**
   - `config_test.go:525-547` pins this with the same whole-struct comparison. The "a local value wins" row changes `ReviewMaxBounces` from the template's 3 to 2, so it is not vacuous.

3. **The `template defaults` subtest pins the template's parsed values per field.**
   - Pinned by `config_test.go:163-176` itself.
   - The map has the same field-and-value pairs as the removed `templateConfig()`.
   - A mis-parsed pinned field fails through `DeepEqual`.
   - A renamed `Config` field fails through the `IsValid` check.
   - An entry that is an empty or zero value, such as `DiscussionInteractive: false`, is still checked.
   - Fields the old literal left at their zero value were pinned only implicitly, because the whole-struct compare caught a non-zero load. Examples are `PRReviewAskParent`, `Seed`, `DiscussionAdvisors` and the judge keys. They are now unpinned, which matches the Idea ("pin only the fields it changes").

4. **`TestModules_EveryTemplateKeyClassified` is the one test pinning each module's whole key set.**
   - Pinned by its own `slices.Equal` checks (`configreg_test.go:173-197`) and by `coveredByEntry` and `holdsPath`.
   - The behaviour is unchanged, so the existing assertions still pin it.

5. **The new failure messages name the key and the map or list to edit.**
   - **Unclassified-unit message** (`configreg_test.go:165`) and **stale-entry message** (`:170`): none.
   - **`specFunc` derivation** (`runtime.FuncForPC(reflect.ValueOf(m.Spec)...)`, `:158`): none. The messages only print it. Nothing checks that it resolves to a named function rather than `""` or a closure name.
   - **`keyListWayForward`** (`:204-216`): none. Nothing checks:
     - the add and drop directions (`keysOutside(want, got)` gives keys in `got` but not `want`, the ones to add to the expected list);
     - the order-only `sort … in this test` branch;
     - the combined add-and-drop string.
   - **`keysOutside`** (`:219-227`): none.

**Findings**

- **F1, LOW, `consistency`, `internal/configreg/configreg_test.go:204-227`.**
  - `keyListWayForward` and `keysOutside` are new decision logic with three output branches: add, drop, and sort-only.
  - No test exercises them, and the real test reaches them only when a classification is already wrong. A flipped direction or a wrong list name would go unnoticed.
  - Evidence: the file's test list has no test of either helper (`:24-443`).
  - Fix: add a small tier-0 table test `TestKeyListWayForward` with rows for add only, drop only, both, and order-only.
  - A separate test of the helper is justified here only because the public test cannot reach these branches without a broken registry; if the maintainers do not want tests of test helpers, this is a NIT.

- **F2, NIT, `consistency`, `internal/configreg/configreg_test.go:158`.**
  - `specFunc` is trusted for every module's `Spec`.
  - The `Spec` values in `internal/configreg/configreg.go:183-205` are all plain package functions, so the names resolve.
  - A nil `Spec` gives `""` without panicking, because `(*Func).Name` tolerates a nil receiver, but the message would then read "returns" with a blank name.
  - Nothing pins this. I would not require a test.

**Tool calls**

None; the inherited context already held the diff, both test files and `configreg.go`'s registry lines.
