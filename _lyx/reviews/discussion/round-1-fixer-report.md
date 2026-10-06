# Round 1 fixer report

All six findings fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged: the gate appends its own round entries.

## Changed

- F1: Decision 9 now runs the rerun once per in-scope package into separate scratch files, states why (the base tool takes one go pattern and rewrites `-out` whole), and makes the section splice a step this task does outside the tool.
- F2: Decisions 2, 3 and 5 point to where a reason lives (the directive from Decision 10, or the PR body for an unmade tier move).
  Decision 2 adds a bound: the PR body lists the receiving covering test for each deleted candidate whose assertion moved, and a deletion without one is not allowed.
- F3: Decision 10 says the criterion is a comparison of the final rerun's lists with the source directives, and that the prune repeats until the rerun lists only annotated candidates, a test with nothing to pin being deleted or folded.
  The first acceptance criterion says "final rerun" and "checked by comparing".
- F4: Decision 1 gains a bound for a needed core or testkit edit: leave it, list it in the PR body, keep the named test.
  The last acceptance criterion is scoped to files outside core and testkit, and a new criterion covers the listed references.
- F5: Decision 12 names `-coverpkg=./...` and ties an explained drop to deleted `Removable: yes` candidates covered elsewhere; any other drop is restored.
- F6: Decision 6 defines an unused helper as one no file under any tag, `llm` included, calls.

## Deferred

None.
