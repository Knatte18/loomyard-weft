# Support log — test-tiers

## Interview

Autonomous mode: no operator answers; every decision below is a self-pick recorded in the question ledger.

1. Read the board entry `test-tiers` (with issue #421) and the related entries `test-suite-measure`, `card-batching`, `shuttle-turns` and `baseline-drift-gate`.
   The entry carries the hub's proposed gate design (2026-10-05), the after-state and budget asks, the cgo link-cost ask, the comment-lint options, and #421.
2. Explored the gates: `loomshed/verifygate.go` runs the plan verify at the Webster-Burler round, `websterengine/verifygate.go` at Webster's gate, `landingshed/verifygate.go` at Publish and Finalize, all through `verifytree.Verify`, which keeps one record per worktree.
   PR-Rework routes through Plan-Bouncer and Webster again, so the only round gate is Webster-Burler's.
3. Read `websterengine/cardgate.go`: the card gate is already a Go-derived command (`go build ./... && go test ./...` plus the card's own integration packages), so a Go-derived round command has a precedent.
4. Read the plan spec's verify model, the Plan-Write template and the plan-review rubric: they let `## verify:` run the `tmux` tier and defer the default to this task.
5. Measured the tree-sitter reach: 35 packages link it in their test deps; `quarry/glyph` is cgo-free, and tree-sitter enters through `quarry/quarry`, imported by `planglyph`, `websterengine` and `quarrycli`.
6. Checked the line-break lint options: the Webster master already loads `scribe:prose` (`websterengine/roles.go`) and its forks inherit it, so option 2 is in place and has not worked; option 1, the lint, is needed.
   `tools/godocreflow` holds reusable splitting heuristics.
7. Read the timing doc's before-state and post-retag sections for the after-state method, and noted the `tmux` tier's failing `TestSmokeDotFillCrossClientControl`.
8. Settled the design: D1–D11 in the decision record.

## Rejected alternatives

- **Round gate runs only the changed packages.** Too narrow: a fix in `fabricengine` can break `fabriccli`.
- **Round gate base is the round's own start commit.** A crash, a skipped gate or a second round would leave earlier changes outside the set; the last full pass makes the set a function of the tree.
- **A second plan section (`## verify-round:` or `## verify-landing:`) written by Plan-Write.** Puts a repo-level fact into every plan and relies on the author to keep the impacted set right; the Go derivation is mechanical, and the `tmux` tier at Publish is a repo fact, so landing config fits.
- **Running the `tmux` tier in the plan verify.** Then Webster's gate pays about 230 s for every PR-Rework too; the operator asked for it at Publish only.
- **Running `publish_verify` at Finalize too.** Finalize's merge-in is covered by the plan verify; the `tmux` tier ran at Publish on the PR's tree.
- **Pinning wall time in the budget guard.** Wall time moves with machine load (67–90 s versus 60 s idle), so it would fail on load, not on growth; reported only.
- **Failing the budget guard when a count falls below budget.** Forces a budget edit for every prune with no gain; under-budget passes.
- **Loading or inlining `scribe:prose` in the implementer stencil.** Already loaded by the master and inherited by forks; three runs still ended unconverged on line breaks.
- **A whole-repo comment lint as a guard test.** It would flag legacy wrapped comments the rule exempts; the lint must be diff-scoped.
- **A lint that also flags compound-predicate comma breaks.** Telling them from independent clauses needs a parser; a false positive would block a correct comment, so those stay review findings.
- **"The clause-boundary break is required" for #421.** Nobody can apply "long sentence" consistently, and its absence would become a perennial review finding.
- **Package-level guard marker.** Would run heavy packages like `cmd/lyx` whole every round; a per-test marker runs only the guards.
- **A separate tier for heavy packages to absorb the contention flake.** The narrower round gate already lowers contention; no measured case needs a new tier.
- **Growing the allowed tree-sitter list instead of cutting the chain.** Leaves the link cost the operator asked to remove; kept only as the fallback in Open risks.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Which gate runs which tier?** Resolved (auto-pick): the hub's proposed design, D1; tmux at Publish via config, LLM never.
- **Q2 — How does the round gate know what changed?** Resolved (auto-pick): diff from the commit of the last plan-verify pass, kept in per-command verified-tree records (D2, D3).
- **Q3 — What about non-Go target repos?** Resolved (auto-pick): no `go.mod` at the root falls back to the full plan verify; the lint passes vacuously.
- **Q4 — Where does the `tmux` tier run, and how?** Resolved (auto-pick): landing config `publish_verify`, empty by default, set to `go test -tags tmux ./...` in loomyard (D4).
- **Q5 — The failing `tmux` test blocks Publish.** Resolved (auto-pick): making the tier green is in scope; flagged in Open risks.
- **Q6 — How are guard tests found?** Resolved (auto-pick): a per-test `//lyx:guard` marker; binary-building tests found mechanically from `lyxbin` imports (D2, D5).
- **Q7 — Budget: test count only, or wall time too?** Resolved (auto-pick): count pinned, wall time reported (D9).
- **Q8 — Lint or stencil change for comment line breaks?** Resolved (auto-pick): the lint, diff-scoped, in the card and round gates (D7).
- **Q9 — #421: is the clause-boundary break required?** Resolved (auto-pick): permitted, not required; non-clause comma breaks are violations (D6).
- **Q10 — Is `scribe:prose` updated to match?** Open for the operator: it lives in the scribe plugin outside this repo; noted in Open risks.
- **Q11 — Where exactly is the tree-sitter chain cut?** Delegated to Plan-Write after the link-time measurement, with the goal and guard fixed (D8).
- **Q12 — New PATTERN entries?** Resolved (auto-pick): none; the operator wanted the Testing section revised, not grown, so `PATTERN-test-speed`, `PATTERN-verified-tree` and `PATTERN-comment-line-breaks` absorb the changes (D11).
- **Q13 — The `reedengine` contention flake.** Resolved (auto-pick): out of scope; no measured case for a heavy-package tier.
