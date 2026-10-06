# Card 12 — shed-prune

**Edit:**
- `internal/shedengine#`
- `internal/shedrecipe#`
- `internal/shedrun#`
- `internal/shedtransient#`
- `internal/shedverbs#`
- `internal/shedcli#`
- `internal/shedcheck#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the shed packages' test files.
`internal/shedadapters` is core's and stays untouched (Decision 1).

`internal/shedengine` and `internal/shedrun` carry large candidate sets of state-transition and run-id checks, the main fold leads.
`PATTERN-shed-verb-set` and `PATTERN-shed-run-directory` tests keep one row per verb and per run-id form; `PATTERN-transient-stop` tests keep the boundary where the mark is set.
`internal/shedcli`'s `integration` files (`parity_test.go`, `seed_integration_test.go`) hold this group's tier-move and scenario-merge leads.

**ImpactSummary:** Test files of seven shed packages only; no production change.

**Commit:** 12: Prune shed tests
