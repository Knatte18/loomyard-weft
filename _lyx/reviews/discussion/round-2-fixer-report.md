# Round 2 fixer report

## Changed

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D6 now says a successful `reset` and `run --fresh` clear the persisted pre-fix head, and that an earlier gate's value is otherwise used as recorded.
- F2: D3's bound now requires the rewrite to leave allow and deny matching unchanged, pinned by a test, with the fallback applying if it cannot.
- F3: R4 now calls the by-path residual an auto-pick (Q13); D6's Bound states the residual.
- F4: D5 states the condition that `run` re-enters Master for a run at its exit audit, verified by the Plan producer.
- F5: four acceptance criteria added: failed tool results are not evidence, the post-accept way forward, the pre-fix head lifecycle, and the D3 matching test.

## Deferred

Nothing deferred.
