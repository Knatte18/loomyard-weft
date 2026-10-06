# Card 11 — loom-prune

**Edit:**
- `internal/loomengine#`
- `internal/loomrecipe#`
- `internal/loomshed#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `internal/loomengine`, `internal/loomrecipe` and `internal/loomshed`.
`internal/loomcli` is core's and stays untouched (Decision 1).

`internal/loomengine` has the second-largest candidate set and several `Removable: no` entries; its checks over friction notes, status and plan generations are fold leads.
`PATTERN-plan-generation` and `PATTERN-friction-capture` tests keep their archive and note-content assertions.
`internal/loomrecipe`'s recipe-row checks keep one row per producer, and the perch wiring checks (a `Bouncer` row and its `Burler` partner) keep both directions.
`internal/loomshed`'s `verifygate_integration_test.go` is its tier-move lead; a verify-gate test that runs a real verify command stays tagged (`PATTERN-verified-tree`).

**ImpactSummary:** Test files of three loom packages only; no production change.

**Commit:** 11: Prune loom engine, recipe and shed tests
