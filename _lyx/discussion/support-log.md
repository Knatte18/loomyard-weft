## Interview

Autonomous mode: no operator answered, so every question below was answered by auto-pick after reading the board and the code.

1. Read the task, `test-suite-prune-core`, `test-tiers` and `test-suite-measure` on the board.
   The three tasks split the work: core prunes six packages plus `internal/testkit`, this task prunes the rest, and `test-tiers` owns the after-state and the budget guard.
2. Read `9cd12fd19` (test-suite-measure) and the generated `docs/benchmarks/test-redundancy.md`.
   The report covers the `integration,tmux` tags.
   In-scope packages hold 826 `Removable: yes` candidates and over 3400 tests; `cmd/lyx` alone has 32, most of them guard self-checks.
3. Package boundary: are sub-packages of core directories in scope?
   Auto-pick: yes, since the board lists packages.
4. Shared files: who owns `internal/testkit` and `test-redundancy.md`?
   Auto-pick: testkit is core's; each task regenerates only its own sections of the report.
5. Where do the stated reasons for kept candidates live?
   Auto-pick: the PR body, which is the squash commit message.
6. How to treat guard self-check candidates in `cmd/lyx`?
   Auto-pick: keep them, because they pin that the guard fires, which coverage cannot see.
7. Folding scope: candidates only, or every package?
   Auto-pick: every in-scope package, as the board's Clean list says.
8. `llm` tests?
   Auto-pick: out of scope.
9. "failed" entries in the report?
   Auto-pick: re-run each alone; fix it if it fails alone, otherwise record it as a tool artifact.
10. Measurement and coverage method.
    Auto-pick: `testtiming -tags integration,tmux`, median of three, on the base commit and the final tree; per-package `-coverprofile` before and after.

## Rejected alternatives

- **Treat `internal/reedengine/render` and `internal/shuttleengine/claudeengine` as core's.** Rejected: the board names the six packages, and Go packages are not directory trees.
- **Regenerate the full `test-redundancy.md`.** Rejected: in this worktree the core packages are pre-prune, so a full rerun would overwrite core's sections with stale results and cause a conflict.
- **Change `cmd/testtiming` here to read a keep comment.** Rejected: `test-suite-prune-core` owns that tool change (its D2), and this task is judged against the tool at its base.
- **Reasons only in the PR body (round-1 Decision 10).** Rejected at parent review: after both tasks land, core's tool would show every kept candidate as unexplained.
- **Fold only candidates and their covering tests.** Rejected: the board asks for folding in every package; scoping it down would be an unasked deferral.
- **Add new fakes to `internal/testkit` for tier moves.** Rejected: testkit is core's in this split, and duplicating fakes breaks `PATTERN-testkit`.
  Such a move is reported to the hub instead.
- **Delete `Removable: no` candidates when their partner is kept.** Rejected: the removable set is the only set the report proves safe to delete together.
- **Take before-state timing from the report.** Rejected: those runs were coverage-instrumented, one test at a time.
- **Prune `llm` tests too.** Rejected: they are unmeasured, and verifying a change would spend tokens.

## Review rounds

_No rounds yet._

### Parent review, round 1

- F1 (BLOCKING, consistency): reasons only in the PR body clash with core's `//testtiming:keep` directive. Fixed.
  Decision 10 now puts a `//testtiming:keep <reason>` above each kept candidate, in core's D2 spelling, with no tool change.
  The first acceptance criterion now checks the directive.
  The PR-body durability assumption is dropped.
  Open risks gain one line: core's D1 will add new candidates in these packages after landing, and those are not this task's.
  Decision 9 and the scope line about the tool are unchanged.

## Question ledger

- Q1 Sub-packages of core directories in scope? — auto-pick: yes (Decision scope). Resolved.
- Q2 `internal/testkit` ownership? — auto-pick: core's, including the kits' own tests. Resolved.
- Q3 Report regeneration without conflicting with core? — auto-pick: section-scoped replacement (Decision 9). Resolved.
- Q4 Where are kept-candidate reasons stated? — round 1 auto-pick was the PR body; parent review F1 changed it to `//testtiming:keep <reason>` on the test, in core's spelling (Decision 10). Resolved.
- Q5 Guard self-check candidates? — auto-pick: kept with reason or folded (Decision 3). Resolved.
- Q6 Folding breadth? — auto-pick: every in-scope package (Decision 4). Resolved.
- Q7 Tier moves needing a new shared fake? — auto-pick: not made, reason stated, hub told (Decision 5). Resolved.
- Q8 `llm` files? — auto-pick: out of scope. Resolved.
- Q9 "failed" report entries? — auto-pick: diagnose, fix if failing alone (Decision 8). Resolved.
- Q10 Measurement and coverage method? — auto-pick: Decisions 11 and 12. Resolved.
- Q11 Unused helpers and empty files after deletion? — auto-pick: deleted; `TestMain` kept (Decision 6). Resolved.
