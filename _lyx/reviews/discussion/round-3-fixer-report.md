# Round 3 fixer report — discussion

## Changes

- **F1** (fixed as proposed): the premise holds; the hub's `_board/_lyx/config/landing.yaml` sets `deploy: "./update-plugins.sh"`, so D17's deploy-key-gated comparison never ran there.
  D17 now names two next-wake board checks independent of the `deploy` key: a running task whose board entry is `done`, or whose pair is gone, is treated as landed and gets the newly-ready check, and a ready, unstarted task is started.
  The `origin/main` versus `origin/prod` comparison is kept as a separate deploy-recovery check, scoped to an empty `deploy` key and to landings into a `require_pr_to_base` branch.
  D16 now points at D17's board checks for the landing and keeps `origin/prod` as the deploy's baseline only.
  The acceptance row on the fallback is split: one row pins the board-done check, and the other is narrowed to the deploy-recovery check.
- **F2** (fixed as proposed): D11's heading, its first sentence, its plan-listing sentence and the Scope bullet now read "Info-or-above line, Warn included, in a loop or polled path".
  The exemption sentence drops "keeps Warn and", and the rationale states that the audit narrows the board's ask to Warn calls in a loop or polled path.

## Disputed

None.

## Deferred

None.
