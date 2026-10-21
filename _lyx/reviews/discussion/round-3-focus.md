---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md` D8 and D13: the implementer fork runs each card `**Verify:**` line itself in a foreground Bash call (`contracts/stencils/webster/webster-body-implementer.md` step 5), and the shell guard's `raw-go` row lets a package-scoped `go vet` or `go build` through. Settle whether the record makes that run hold one slot with `-p` and `-trimpath`, or refuses such a line at the plan gate, or amends the declared `PATTERN-gate-slot` text to name it with a bound."
  - "In D13 and the declared `PATTERN-gate-slot` and `PATTERN-test-tiers` texts: `cmd/testtiming` has a default timing mode, a `-redundancy` mode and a `-resources` mode, and only `-resources` takes a slot today (`cmd/testtiming/resources.go`). Settle whether the record decides each mode's slot status and whether the declared texts match that decision."
  - "In D8 and `internal/loomshed/publishfailure.go`: the confirm-the-fix pass joins failing-test runs and a package pass to the round command with `&&`. Settle whether the record says each step is its own slotted `lyx gate test --wait` call, and whether D8's claim about every command `verifytree.Verify` runs agrees with that."
  - "In the Acceptance criteria and D3: under the `none` scoper a computed scope is whole, under the `go` scoper a `.md`-only diff yields an empty set. Settle whether every acceptance criterion about a `.md`-only diff names the scoper and agrees with D3."
---

The focus names sites to check; it caps no severity and pre-states no verdict.
