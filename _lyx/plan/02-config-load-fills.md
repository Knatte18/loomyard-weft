# Card 2 — config-load-fills

**Edit:**
- `internal/configengine/config.go`
- `internal/configengine/config_test.go`
- `internal/loomengine/config_test.go`
- `internal/loggerconfig/config_test.go`

**Create:**
- `internal/configreg/fill_test.go`

**Prosa:**
- `docs/shared-libs/configengine.md`
- `CONSTRAINTS.md`

**Uses:**
- `plan:internal/yamlengine#FillMissing`
- `internal/yamlengine#MissingKeys`
- `internal/logger#Info`
- `internal/configreg#Modules`

**Intent:**
Make `configengine`'s shared `load` body fill a present config file's missing template keys from the template in memory instead of refusing (D1, fixes #339).
This lets a key a deploy adds load at its template default with no `lyx config reconcile` sweep.
It never writes the config file.

In `load`, between the read and `envsource.Build`:

1. Call `yamlengine.FillMissing(template, fileBytes)`; an error (a shape mismatch, unparseable YAML) returns `config file <path>: <err>`, naming the key-path.
2. When it filled any key, log one `logger.Info` line (`configengine: filled missing keys from template`) carrying `module`, `file` and the comma-joined key-paths.
3. Run `yamlengine.MissingKeys(template, filled)` as the backstop: what it still reports is a key missing inside an element of a present list, which `FillMissing` never fills; refuse with `config file <path>: missing keys: <paths>`, dropping the `lyx config reconcile` hint, since reconcile carries lists whole and cannot add the key either.
4. Resolve `filled` through `envsource.Build` and `yamlengine.Resolve` exactly as before, so a filled `${env:NAME}` with `NAME` unset still refuses.

`Load` and `LoadOrTemplate` both get this through the shared body; the absent-`_lyx/` and absent-file branches, and `Load`'s absent-file error with its `lyx config reconcile` hint, stay as they are.
Update `Load`'s and `load`'s doc comments so neither says a missing key refuses.

What this lets through, and its bound: a present file lacking template keys now loads where it refused.
Only key-paths in the module's embedded template are filled, only with the template's own value, only where the file lacks the key and every ancestor it holds is a mapping;
a shape mismatch, a list-element miss, unparseable YAML and an unset required env marker still refuse.

Tests:

- `internal/configengine/config_test.go`: flip `TestLoad_MissingKey`, `TestLoadOrTemplate_PresentMissingKey` and `TestLoadOrTemplate_PresentEmpty` to assert the load succeeds with the template default, the file on disk is byte-identical afterwards, and the captured log (via `logger.SetOutput` plus `logger.SetVerbosity(1)`, restored in `t.Cleanup`) holds exactly one fill line naming the module and key-path.
  Add fixture-template cases: an extra key surviving beside a filled one; a present empty string and an emptied list (`require_pr_to_base: []`) kept; a filled `${env:NAME}` with `NAME` unset refusing; a null where the template holds a mapping, a mapping where it holds a scalar, and a key missing inside a present list element each refusing with the key-path named; a complete file logging no fill line.
- `internal/loomengine/config_test.go`: `TestLoadConfig_MissingFrictionKeys` becomes the positive case: the friction keys load at their template defaults.
- `internal/loggerconfig/config_test.go`: `TestLoad_MissingKeyErrors` becomes the positive case: the missing keys load at their template defaults.
- New `internal/configreg/fill_test.go`: for each of `fabric`, `board`, `landing`, `loom` and `reed`, from `Modules()`' real template, drop one top-level key, load it through `configengine.Load` from a temp `_lyx/config/`, and assert the loaded value equals the template's resolved default, the file is byte-identical, and one fill line names the module and key-path.
  For `reed`, also drop `selvage.height_rows` alone and `status_line` whole, asserting each loads with its defaults.
  An empty but present file loads as the template.

Fix any other test `go test ./...` surfaces that asserted the old refusal the same way, as a positive case.

Docs in this commit:

- `docs/shared-libs/configengine.md`: flow step 3 describes the fill, the Info line and the list-element backstop; Key properties say a missing key resolves to its template default and is logged, and the strict-on-`Load` bullet drops missing keys from its list; the error table's missing-keys row describes the list-element refusal without the reconcile hint.
- `CONSTRAINTS.md`, Config Strictness Invariant (D7): add the bullet "A key missing from a present config file resolves to its template default and is logged; `Load` and `LoadOrTemplate` differ only on an absent `_lyx/` or file."

**ImpactSummary:** Every module loading through `configengine.Load` or `LoadOrTemplate` now accepts a present file with missing template keys; tests asserting the old refusal flip to positive cases.

**Verify:** go test ./...
