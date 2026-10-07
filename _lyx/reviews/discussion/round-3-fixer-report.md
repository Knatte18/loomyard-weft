# Round 3 fixer report

Every finding of `round-3-review.md` is fixed in `_lyx/discussion/decision-record.md`; F1 also touched `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1 (BLOCKING, design).**
  D4 now says `landing` is per-worktree and landing never carries it to the parent, so the committed `publish_verify` value reaches `main`'s `landing.yaml` only through the hub's re-apply after Finalize's notice.
  It says a copy without the key loads as the template's empty value, so a task forked earlier runs no `tmux` tier at Publish.
  The "everything reaching `main`" sentence now reads as a condition on the task's own copy.
  D4's Bound says "empty or absent".
  A new Open risk records the rollout gap, and a Notes-for-the-plan-writer line asks the PR description to state the re-apply.
  Q14 in the support log carries the same fact.
- **F2 (MEDIUM, consistency).**
  D2's bound now says a miss is caught at Publish, or at Finalize where Publish is a no-op, and that a landing into a parent outside `require_pr_to_base` runs no `tmux` tier.
- **F3 (MEDIUM, consistency).**
  D2 and D5 no longer say "one rework round": the stop halts the run Stuck and is fixed forward.
  D5 says the plan verify runs the untagged and `integration` tiers, and that a scanning test in a `tmux` file is caught only where `publish_verify` runs.
- **F4 (MEDIUM, design).**
  D7's Scope checks a break only when it is new in the diff (its two words not adjacent across a line end in the base text), and a line end an edit carries over is not new.
  The Finding bullet follows, D6's rewrap bullet names the carried-over case, and an acceptance criterion covers fixing one flagged break leaving the block's other breaks unflagged.
- **F5 (MEDIUM, design).**
  D7 narrows the lint-gap sentence to the two no-base cases and adds the Webster-gate fixer-fork commits as a second place no gate lints, with the same bound.
- **F6 (LOW, consistency).**
  D6 names the exact clause of the PATTERN bullet it replaces, keeps "a line-width finding is never legitimate" and "the rule governs new and changed comments only", and rewords the first bullet to make the clause break permitted.
- **F7 (LOW, consistency).**
  The Goal no longer says the full plan verify runs "only where a tree is about to land".
  D1's last row says "Every gate loomyard configures" never runs the `llm` tier.
  D2's sentence follows.
- **F8 (NIT, consistency).**
  D6's colon bullet is merged into the sentence-end bullet.

## Deferred

None.
