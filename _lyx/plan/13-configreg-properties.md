# Card 13 — configreg-properties

**Edit:**
- `internal/configreg/configreg_test.go`

**Create:**
- `internal/configreg/registration_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`
- `internal/configreg/configreg.go`

**Intent:** Assert the config registry by property instead of a literal list (D6).
`TestNames` asserts that `Names()` is sorted, unique, and that every name's `Template` resolves to a non-empty template, with no literal name list;
`TestModules_SeedOnly`, `TestTemplate_Found` and `TestTemplate_NotFound` stay, since each pins a specific module's attribute.
`registration_test.go` is the independent membership oracle, on `scankit`: it walks production files under `internal/` for a package-level `ConfigTemplate` declaration and parses `configreg.go`'s `Modules()` for every `<pkg>.ConfigTemplate` it references, and fails when a declaring package is not registered or a registered reference names no declaring package.
It has a floor.
Prove it by removing one entry from `Modules()`, watching the scan fail, and restoring it;
name the proof in the commit body.
What this lets through, bounded per D6: the property test no longer notices a module dropped from `Modules()`, which the registration scan catches, and a new module's `ConfigTemplate` is caught there with no test edit.

**ImpactSummary:** `internal/configreg` tests only; one literal-list assertion becomes properties plus a source scan.
