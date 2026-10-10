# Round 2 fixer report: discussion

All four findings were validated against the code and fixed.

## Fixed

- **F1** (MEDIUM, consistency): confirmed in `internal/loomcli/resume.go`, where a run with no live driver passes the merge guard and then always refuses with `noLiveDriverMessage`.
  D3's Decision and the fifth acceptance criterion now split the verbs: `lyx loom start` proceeds for a run halted there or `running` there with a dead driver; `lyx loom resume` proceeds when a parked driver is live, and with no live driver reports the no-live-driver refusal instead of `merge_in_progress`.
- **F2** (MEDIUM, design): confirmed in `hubgeom.decideParent`, which forms `<shortname>:<slug>:orch` when the origin's `ParentWorktree` is not the prime.
  D6 now refuses both labels when the strand name is non-empty and differs from the run's recorded parent name from `hubgeom.ResolveParent`, so it no longer assumes a slug-free parent.
  The refusal also says that the operator's unnamed shell may always run the verb.
  Rationale, Bound, the auto-mode assumption, the acceptance criterion, the `PATTERN-agent-name` constraint and the plan-writer note were updated to match.
  Open risks now carries the `orch-any-worktree` dependency.
  In the support log, the slug rule is a rejected alternative and the ledger entry is updated.
- **F3** (LOW, design): confirmed in `verifyGate.recordFailure`, which logs a failed `WritePublishFailure` and returns while `verdict` still gives the failed-verify reason.
  D1 now names the goto only when the record was written, and the record writer reports whether it wrote.
  The Goal, Scope In, the first acceptance criterion and the `PATTERN-verified-tree` constraint were updated to match.
- **F4** (LOW, design): the conflict session is its own shuttle strand (`mergeresolve`, role `conflict`), so it can outlive the driver.
  Before its abort, D2's row now stops any live conflict-session strand of its own run through shuttle's stop verb, and D2's Bound bounds that stop.
  D3's Bound names this, and Scope In, a `PATTERN-shuttle-stop` constraint, the third acceptance criterion and a plan-writer note were added.
  Two support-log ledger entries record the F3 and F4 picks.

## Disputed

None.

## Deferred

None.
