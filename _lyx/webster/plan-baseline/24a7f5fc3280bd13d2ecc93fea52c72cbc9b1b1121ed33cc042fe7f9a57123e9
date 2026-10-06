# Card 14 — reed-prune

**Edit:**
- `internal/reedcli#`
- `internal/reedengine/render#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `internal/reedcli` and `internal/reedengine/render`.
`internal/reedengine/render` is a separate package from core's `internal/reedengine`, whose own files stay untouched (Decision 1).

`internal/reedcli` is the slowest in-scope package, almost all of it in its `tmux` files: tests that each start a tmux server and differ only in the strand steps after it are the scenario-merge leads, merged only within the `tmux` tier.
Its `TestSmokeDotFillCrossClientControl` is a `failed` entry (Decision 8).
Each merged family is named in the commit message with the server starts it removed.
`internal/reedcli` has `llm` files that Decision 6's helper rule covers.

**ImpactSummary:** Test files of two reed packages only; no production change, and core's `internal/reedengine` files are untouched.

**Commit:** 14: Prune reed CLI and render tests

**Verify:**
go test -count=1 -tags tmux ./internal/reedcli/
go vet -tags llm ./internal/reedcli/
