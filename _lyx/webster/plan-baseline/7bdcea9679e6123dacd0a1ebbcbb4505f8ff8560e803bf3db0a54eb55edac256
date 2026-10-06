# Card 20 — stencil-state-prune

**Edit:**
- `internal/stencil#`
- `internal/stencilcli#`
- `internal/stencilstore#`
- `internal/state#`
- `internal/cliwire#`
- `internal/agentname#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the stencil packages, `internal/state`, `internal/cliwire` and `internal/agentname`.

`internal/stencil` has a large candidate set with many `Removable: no` entries, which are folded or kept, never deleted (Decision 2); a check on rendered stencil text is a unique assertion unless its covering test pins the same text.
`internal/cliwire`'s `specsseed_integration_test.go` and `internal/stencilcli`'s `cli_integration_test.go` hold this group's tier-move and scenario-merge leads.
`internal/agentname` tests keep one row per name form (`PATTERN-agent-name`).

**ImpactSummary:** Test files of the stencil, state, cliwire and agentname packages only; no production change.

**Commit:** 20: Prune stencil, state and agent name tests
