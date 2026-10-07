# Round 2 fixer report

Every finding of `round-2-review.md` is fixed; nothing is deferred.

## Changed

All edits are in `_lyx/discussion/decision-record.md` unless named otherwise.

- **F1 (BLOCKING):** D4 no longer claims the changed `publish_verify` is in the PR diff.
  It states the value lives on the task's weft branch, is not carried to the PR or the parent, is seen by the hub only in Finalize's notice after the landing, and that an emptied value drops the `tmux` gate for that landing.
  The gap is accepted and the D4 Bound line matches; a new Open risks entry records it.
  `support-log.md` Q14 is corrected the same way.
- **F2:** D4 states `publish_verify` runs only where Publish runs, that Publish is a no-op for a parent outside `require_pr_to_base`, and that Finalize never runs it, bounded by `main` being in loomyard's list.
  "once per landing" became "once per pull-request landing".
- **F3:** D2's closing bound is split: a `tmux`-only failure is caught at Publish where `publish_verify` runs the tier, and an `llm`-only failure passes every gate.
- **F4:** D7's Scope says only a break the edit touches is checked and that a break between two untouched lines is never flagged; D6 gains a bullet making the same statement the replacement for the PATTERN bullet "untouched lines are never rewrapped".
- **F5:** D7 drops the "every card passed it" premise and states that a fallback round's comments are linted by no gate and reach only the reviewers.
- **F6:** D6 gains a bullet defining a sentence end with closing punctuation.
- **F7:** D5's Bound names Publish and Finalize, not Webster's gate.
- **F8:** D8's `go list` runs under `-tags integration,tmux,llm`, and D10's count uses the same tag set.
- **F9:** D5 names the enforcing tests of `PATTERN.md`'s `(test)` entries as the starting set.

No content moved to `support-log.md`, so no requirement left `decision-record.md`.
Each changed guard or fallback (the `publish_verify` edit, the no-base lint pass, the tmux and llm tiers left uncompiled by tests) states what it lets through and what bounds it.

## Deferred

None.
