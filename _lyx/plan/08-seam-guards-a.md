# Card 8 — seam-guards-a

**Edit:**
- `internal/agentname/seam_enforcement_test.go`
- `internal/battenrecipe/seam_enforcement_test.go`
- `internal/battenshed/seam_enforcement_test.go`
- `internal/cliwire/seam_enforcement_test.go`
- `internal/cliwire/bannedecl_enforcement_test.go`
- `internal/cliwire/callerset_enforcement_test.go`
- `internal/frictionengine/seam_enforcement_test.go`
- `internal/landingshed/seam_enforcement_test.go`
- `internal/loomrecipe/seam_enforcement_test.go`
- `internal/loomshed/seam_enforcement_test.go`
- `internal/mergeresolve/seam_enforcement_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the first half of the seam guards to `scankit` per D2 and D3, following the overview's `scan-conversion-protocol`.
Each import guard stays in its own package as a short caller of `scankit`'s import-allowlist assertion with its allowlist beside it, one entry per line (D8).
A guard with checks beyond its import allowlist keeps them on the harness walk and parse: `cliwire`'s banned-declaration and `standalonestate.Derive` caller-set scans keep their AST matching, and any resolver-token or deny rule a seam file carries stays verbatim.
Every converted scan gains a floor where it had none and stale detection on its allowlist.
Prove each guard by one planted violation, named in the commit body with the plant removed.

**ImpactSummary:** Test-only; the seam guards of these packages change scaffolding, verdicts unchanged.
