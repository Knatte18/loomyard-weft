# Card 13 — review-gates-prune

**Edit:**
- `internal/burlercli#`
- `internal/burlerengine#`
- `internal/treadleengine#`
- `internal/parentreview#`
- `internal/preflight#`
- `internal/preflightshed#`
- `internal/verifyrun#`
- `internal/verifytree#`
- `internal/statuscommit#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the review, gate and verify packages' test files.

`internal/burlercli`'s `TestRunCLI_GroupGuard_OutsideGitRepo` is a `failed` entry (Decision 8).
`internal/burlerengine`'s and `internal/treadleengine`'s round and verdict checks are fold leads; `PATTERN-review-round` tests keep their review-before-fix and judge-verdict assertions.
`internal/verifytree` tests keep `PATTERN-verified-tree`'s dirty-tree refusal and HEAD-tree skip checks.
`internal/burlerengine` and `internal/treadleengine` also have `llm` files: a helper is deleted only when no file calls it under any tag, `llm` included (Decision 6), so the card's own verify vets them under `llm`.

**ImpactSummary:** Test files of nine review, gate and verify packages only; no production change.

**Commit:** 13: Prune review, gate and verify tests

**Verify:**
go vet -tags llm ./internal/burlerengine/ ./internal/treadleengine/
