# Card 27 — drift-retire-verify-gate-key

**Edit:**
- `internal/gateslot/template.yaml`
- `internal/gateslot#goReadGates`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestSeedAdoptGate`
- `internal/gateslot/doc.go`
- `internal/gatecli/doc.go`
- `internal/loomshed/doc.go`
- `internal/testkit/envkit#fixedGateLists`
- `docs/dotnet-hub.md`
- `pattern/PATTERN-gate-slot.md`
- `pattern/PATTERN-test-speed.md`

**Intent:** Remove the `drift-verify` gate key (D14), which no row runs since card 16 dropped the Drift-Verify rows' `gate_key` and verify entry.

The gate template drops the `drift-verify` key and its comment line, `goReadGates` drops it, and the docs, the pattern background files and `envkit`'s fixed gate lists stop naming it; `templateDefaults` and `TestSeedAdoptGate` follow the template.
A hub's `gate.yaml` entry for `drift-verify` is left in place: `gates` is a per-name map whose names the operator owns, and nothing reads that entry any more.
D7's current capture takes its slot from the pool with no gate key (card 9).

**ImpactSummary:** Drops one gate key from gateslot's template and its read list; tests and docs that name it follow; hub files keep an unread entry.
