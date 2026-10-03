# Card 10 — package-local-scans

**Edit:**
- `internal/reedengine/panebin_enforcement_test.go`
- `internal/reedengine/selvagepane_enforcement_test.go`
- `internal/planglyph/chokepoint_enforcement_test.go`
- `internal/planglyph/status_enforcement_test.go`
- `internal/planparser/shape_enforcement_test.go`
- `internal/planparser/shape_test.go`
- `contracts/stencils/citation_enforcement_test.go`
- `internal/shedcli/table_test.go`
- `internal/loomcli/bootstrap_test.go`
- `internal/fabricengine/mergevocab_test.go`
- `tools/sandbox/pathresolve_guard_test.go`
- `internal/gitrepo/noforceadd_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the remaining invariant scans to `scankit` per D2, following the overview's `scan-conversion-protocol`: Pane Binary Resolution and the Selvage confinement (`reedengine`), the planglyph chokepoint and resolve-status tripwires, the Ref-Shape Registry scans and its const-block sync meta-tests' root lookup (`planparser`), the stencil citation guard, the Shed Verb-Set no-`init`/no-`Register` scan (`shedcli`), the Driver Choice tripwire (`loomcli`'s `TestDriverChoiceSingleSiteInvariant_OnlyLoomcliReadsTheSeedDriverField`), the merge-surface Fabric Vocabulary check (`fabricengine`, whose `os.ReadDir(".")` becomes a harness walk of its own package directory), the Dev/Prod Binary Separation guard (`tools/sandbox`) and the Never Force-Add scan (`gitrepo`, whose `os.Getwd()` plus `os.ReadDir` becomes a shallow production-file walk of its own package directory, keeping `noForceAddBannedTokens` and its floor of 5 verbatim, proved by one planted `"-f"`).
Each takes root, walk (shallow where it reads one package directory today), allowlist and floor from `scankit` and keeps its matching logic, exempt sets, pinned site sets and token sets verbatim, including each file's own seeded self-tests.
Every converted scan gains a floor where it had none and stale detection on any allowlist it carries;
`citation_enforcement_test.go`'s existing stale test moves onto the harness's allowlist type.
Allowlists are rewritten one entry per line (D8).
Proofs and stale entries go in the commit body.

**ImpactSummary:** Test-only; the package-local scans change scaffolding, verdicts unchanged.
