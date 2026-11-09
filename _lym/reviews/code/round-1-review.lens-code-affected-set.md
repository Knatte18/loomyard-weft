```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for the callers, tests, docs, rules and messages it forces outside the lines it touches. Per-change results come first.

**Per change, the set found, covered or not**

1. **`templateConfig()` removed, replaced by `templateLoad` (`loadTemplateConfig`)** in `internal/loomengine/config_test.go`.
   - Found: every use inside that file (the table loop, the hub-stack loop, and the `template defaults` subtest).
   - Covered. The `templateConfig` hits outside loomengine are other packages' own helpers.

2. **`template defaults` subtest and `templateDefaults` map.**
   - The map holds the same fields as the old literal.
   - It fails loudly on a renamed field (`FieldByName` invalid → `t.Errorf`).
   - Covered, except finding F1.

3. **`TestModules_EveryTemplateKeyClassified` messages.** I found no other test, doc, stencil or fixture that asserts the old message texts. A grep for the old strings returned only `PATTERN-config-layers.md`'s description, which says the test "fails on a key left out" and is still true.
   - Every `Spec` field in the registry (`internal/configreg/configreg.go:183-205`) is a named package-level `ConfigSpec` function, so `runtime.FuncForPC` yields a stable name. There are no closures or method values, and a nil `Spec` would already fail at `m.Spec()` earlier in the same subtest.
   - Covered.

4. **`keyListWayForward` and `keysOutside` in `internal/configreg/configreg_test.go`.** There are no other callers or tests of these helpers, and none are needed beyond this test. Covered, except finding F2.

5. **Rules documents.** No `Pattern-Departure:` trailers are used in the commit, and the directive declares no PATTERN changes, so nothing is owed there.
   - `PATTERN-test-speed` says a test that reflects over a struct's field set carries `//lym:seam <unit>#<Type>`. See finding F1.
   - `PATTERN-config-layers` and the other PATTERN entries are unaffected.

6. **Change description claims (`_lym/landing/summary.md`).** The summary lists `internal/darnengine/config_test.go` `TestLoadConfig` as breaking on "any key with a non-zero default" via a `templateConfig()` literal. That is right: `internal/darnengine/config_test.go:34-42` builds a literal `Config`. The other listed tests I did not open.

**Findings**

- **F1 — LOW, consistency.** `internal/loomengine/config_test.go:163-176`, the `template defaults` subtest, uses `reflect.ValueOf(templateLoad).FieldByName(...)` on `Config`, and `TestLoadConfig_Loads` carries no `//lym:seam internal/loomengine#Config` marker. `PATTERN-test-speed.md`, "Reflective tests carry `//lym:seam`", asks for the marker when a test body reflects over a struct's field set. This subtest does name-based field lookups, not a walk of the field set, so whether it counts is a judgment call. I found no mechanical scan in `cmd/lym` that detects reflection, so the gate won't catch a missing marker; only review does. The marker also feeds `lym loom validate-plan` (`seam-test-unlisted`), which would require a card that touches `Config` to cover this test. Fix: either add the marker above `func TestLoadConfig_Loads`, or leave it and treat the lookup as not a field-set walk.

- **F2 — NIT, consistency.** `internal/configreg/configreg_test.go:196`: the `MachineKeys` failure text now carries two `so`/`;` clauses ("…so no stray machine file is read by a registered loader; if the classification is meant, …"). It reads awkwardly but is correct, so I'd leave it.

**Tool calls (6 of 6)**
1. grep for the `lym:seam` marker files and the `ConfigSpec` definitions: to see which tests carry the marker and to check how `Spec` is declared.
2. grep in `internal/loomshed/seamtest.go` and `internal/websterengine/config_test.go`: to see how the seam marker is used and enforced.
3. grep in `cmd/lym/*_test.go` for seam tests: to find any scan that detects reflective tests.
4. grep for `NumField`/`reflective`: to confirm no such scan exists.
5. grep for `reflect` in `internal/impactset`: to confirm the marker is only read, not used to detect reflection.
6. grep for the old message strings and `templateConfig`: to find other asserters or users; plus one Read of `internal/darnengine/config_test.go:30-60`: to check the summary's claim about darn's literal.
