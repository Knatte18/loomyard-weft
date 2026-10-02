# Card 19 — reed-shortname

**Edit:**
- `internal/reedengine/geometry.go`
- `internal/reedengine/strand.go`
- `internal/hubgeom/hubgeom.go`
- `internal/standalonegeom/reedgeom.go`
- `internal/reedcli/add.go`
- `internal/reedcli/remove.go`
- `internal/reedengine/strand_test.go`
- `internal/reedengine/contract_integration_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/lock_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`
- `internal/reedengine/naming_integration_test.go`
- `internal/hubgeom/reedgeom_integration_test.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/standalonegeom/naming_integration_test.go`
- `internal/reedcli/naming_integration_test.go`

**Intent:**
Rename the shortname reed is told, and every remaining "code" spelling for it on the reed side, with no behaviour change.

`reedengine.Geometry`'s `NameCode` field becomes `NameShortname`, its comment saying it is the shortname every strand name starts with.
Its two tellers set the renamed field: `hubgeom.ReedGeometry` from the hub's recorded shortname, and `standalonegeom.ReedGeometry` from `agentname.StandaloneShortname(hash8)`, whose comments say a standalone run's shortname is derived from hash8.
In `strand.go`, `validateNaming` and the name former read the renamed field into a local `shortname`, their comments say shortname, and the grammar-guard refusal reads `no strand name can be formed: the told shortname is invalid: %w`; the no-shortname refusal text is card 18's.

reed's help texts name the new forms: `add.go`'s long help reads `<shortname>:<slug>:<role>` and `<shortname>:<role>`, and `remove.go`'s example reads `(shortname:slug:driver)`.

Tests set `NameShortname: "tc"` wherever they set `NameCode: "tc"`, `strand_test.go`'s table field `code` becomes `shortname` with its `MissingCode`/`BadCode` rows renamed to `MissingShortname`/`BadShortname`, and the test names carrying "Code" for this concept say "Shortname":
`TestNaming_EmptySlugGivesCodeAndRole`, `TestReedGeometry_PrimeCarriesCodeOnly`, `TestReedGeometry_UncodedHubYieldsEmptyCode` (to a "has no shortname" wording), `TestReedGeometry_NameCodeIsDerivedFromHash8`, `TestNaming_StandaloneStrandIsCodeAndRoleWithNoParent` and `TestNaming_PrimeStrandHasCodeAndRoleAndNoParent`, with their comments and failure messages following.
Expected values do not change.

**ImpactSummary:** Renames one reedengine Geometry field; its two tellers in hubgeom and standalonegeom, reed's naming code, two reedcli help texts and the reed, hubgeom and standalonegeom tests follow, with no behaviour change.

**Commit:** 19: reed: rename the told name code to shortname
