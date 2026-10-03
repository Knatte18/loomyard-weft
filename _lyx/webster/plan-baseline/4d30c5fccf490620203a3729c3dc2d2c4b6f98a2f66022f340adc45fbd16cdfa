# Card 7 — leaf-guards

**Edit:**
- `internal/buildinfo/leaf_enforcement_test.go`
- `internal/discussionparser/leaf_enforcement_test.go`
- `internal/friction/leaf_enforcement_test.go`
- `internal/githubclient/leaf_enforcement_test.go`
- `internal/gitkit/leaf_enforcement_test.go`
- `internal/gitkit/callerset_enforcement_test.go`
- `internal/modelspec/leaf_enforcement_test.go`
- `internal/pattern/leaf_enforcement_test.go`
- `internal/standalonestate/leaf_enforcement_test.go`
- `internal/summaryparser/leaf_enforcement_test.go`
- `internal/tokenvocab/leaf_enforcement_test.go`
- `internal/gitrepo/internal/gitoracle/leaf_enforcement_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Turn every leaf import guard into a short caller of `scankit`'s import-allowlist assertion, staying in its own package with its allowlist beside it (D3), so `go test ./internal/<pkg>` still catches that package's violation and the `CONSTRAINTS.md` sections naming these tests stay true.
Each allowlist keeps its entries verbatim, written as a one-entry-per-line `[]string` (D8), and an empty allowlist (`buildinfo`'s) stays empty: the check each guard runs today is kept, not tightened.
`gitkit`'s `CopyRepo` caller-set scan moves onto `scankit`'s root and walk, keeps its matching logic and gains a floor.
Prove each guard by one planted import, named in the commit body with the plant removed.

**ImpactSummary:** Test-only; the leaf guards shrink to harness callers with unchanged allowlists and verdicts.
