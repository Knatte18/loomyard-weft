# Card 17 — agentname-shortname

**Edit:**
- `internal/agentname/agentname.go`
- `internal/fabricengine/codebinding.go`
- `internal/fabricengine/clone.go`
- `internal/fabriccli/code.go`
- `internal/reedengine/strand.go`
- `internal/standalonegeom/reedgeom.go`
- `internal/agentname/agentname_test.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/standalonegeom/naming_integration_test.go`

**Intent:**
Rename the name grammar's first segment from "code" to "shortname" inside `internal/agentname`, the package that owns it, with no behaviour change.
The operator finds "code" too vague: it reads as source code and does not say it is the repo's short name.

In `agentname.go`:
- `ValidateCode` becomes `ValidateShortname`, and `StandaloneCode` becomes `StandaloneShortname`; each doc comment opens with its new name.
- The `Name` struct's `Code` field becomes `Shortname`, and every parameter or local named `code` (in `Format`, `Resolve` and the rest) becomes `shortname`.
- The package doc comment, the grammar comments and every error text say shortname: the forms read `<shortname>:<role>` and `<shortname>:<slug>:<role>`, and the grammar error reads `agent name shortname %q is invalid: …`.
The grammar `[a-z][a-z0-9]{1,5}` and both name forms stay as they are.

Callers follow the two renamed functions and the renamed field: `fabricengine/codebinding.go`, `fabriccli/code.go`, `reedengine/strand.go` and `standalonegeom/reedgeom.go` call `ValidateShortname`/`StandaloneShortname`, and `fabricengine/clone.go`'s `CloneOptions.Code` comment points at `agentname.ValidateShortname`.
Only the calls change here; the fabric record and verb rename in card 18 and reed's geometry field in card 19.

Tests: `agentname_test.go`'s `TestStandaloneCode` becomes `TestStandaloneShortname`, its table field `code` becomes `shortname`, and its assertions call the renamed functions;
the two `standalonegeom` tests call `StandaloneShortname` and `ValidateShortname`.
The existing assertions keep their expected values, which is the no-behaviour-change check.

**ImpactSummary:** Renames two exported functions and one field of agentname; its four production callers in fabricengine, fabriccli, reedengine and standalonegeom follow, with no behaviour change.

**Commit:** 17: agentname: rename the name's code segment to shortname
