# Card 6 — lyxcwd-scans

**Edit:**
- `internal/lyxcwd/enforcement_test.go`
- `internal/lyxcwd/docslink_test.go`
- `internal/lyxcwd/raddle_guard_test.go`
- `internal/lyxcwd/leaf_enforcement_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Move `internal/lyxcwd`'s scans onto `scankit` per D2 and D3, following the overview's `scan-conversion-protocol`.
`repoRootForEnforcement` and `walkEnforcementRoots` are deleted;
`TestEnforcement`, `TestEnforcement_GeometryLiterals`, `TestEnforcement_FabricVocabulary`, the docs-link scan and the raddle guard walk through `scankit` instead, each keeping its matching logic, owner sets, allowlists and token sets verbatim.
The docs-link scan keeps its own temp-tree self-tests by passing the fixture tree as the walk's base, and keeps its existing stale-entry report, now through `scankit`'s allowlist type;
`docsLinkAllowlist` becomes one entry per line (D8).
Every converted scan gains a floor where it had none and stale detection on any allowlist it carries.
`leaf_enforcement_test.go` becomes a short caller of `scankit`'s import-allowlist assertion with `lyxcwd`'s allowlist beside it (D3).
`stripGoComments` and its test stay in this package: they are this package's matching logic, not scaffolding.
Proofs and stale entries go in the commit body.

**ImpactSummary:** Test-only; five scans in `internal/lyxcwd` change scaffolding, the shared walker helpers are removed, verdicts unchanged.
