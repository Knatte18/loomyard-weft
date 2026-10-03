# Card 9 — seam-guards-b

**Edit:**
- `internal/orchengine/seam_enforcement_test.go`
- `internal/parentreview/seam_enforcement_test.go`
- `internal/shedbuild/seam_enforcement_test.go`
- `internal/shedengine/seam_enforcement_test.go`
- `internal/shedrecipe/seam_enforcement_test.go`
- `internal/shedtransient/seam_enforcement_test.go`
- `internal/shedverbs/seam_enforcement_test.go`
- `internal/shuttleengine/seam_enforcement_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/treadleengine/seam_enforcement_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the second half of the seam guards to `scankit` per D2 and D3, in the same shape as card 8 and following the overview's `scan-conversion-protocol`.
Allowlist guards become short callers of the import-allowlist assertion with a one-entry-per-line allowlist (D8).
`shedverbs` keeps its resolver-token ban (`TestNoResolverInvariant_NoOSGetwd`) on the harness walk, and `shuttleengine`'s provider-seam rule stays a deny check on `claudeengine` over the harness parse.
`completionsignal_enforcement_test.go` takes its package-directory read from `scankit` and keeps its two AST scans and pinned site sets verbatim.
Every converted scan gains a floor where it had none and stale detection on its allowlist.
Prove each guard by one planted violation, named in the commit body with the plant removed.

**ImpactSummary:** Test-only; the seam guards of these packages change scaffolding, verdicts unchanged.
