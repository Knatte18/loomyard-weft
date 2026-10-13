# Round 4 fixer-report — discussion

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Premise checked: `readySlugs` (`internal/battencli/land.go`) counts any unclaimed task with no `depends_on` as ready, the orch policy (`contracts/stencils/orch/orch-template-role.md`) starts a task only after operator approval, and `rowNotifier.send` (`internal/battenshed/rownotice.go`) appends to the notice log before the hold-back check.
  D17's second dropped-notice bullet now starts only the tasks a landed task's `landed; newly ready:` notice-log line names, with a `Bound:` sentence after the list.
  D16 now says the orch learns of newly ready tasks from that notice-log line.
  Added the acceptance row pinning the bounded fallback.
  Also changed D17's first bullet from "the newly-ready check runs for it" to "the newly-ready check below runs for it", so it points at the bullet that now names what the check reads.
- **F2** (MEDIUM), fixed as proposed.
  Premise checked: `PruneBoard` calls `board.Prune(true)`, `Store.Prune` drops every done entry, and Worktree-Teardown follows Land in `contracts/recipes/batten-recipe.yaml`.
  D16 now says Finalize already marked the entry `done` and the Land row's prune removed it.
  D17's first bullet and the matching acceptance row now read "whose board entry is `done` or gone from the board, or whose pair is gone".
- **F3** (LOW), fixed as proposed.
  D11's exemption sentence now covers any Info-or-above line in no loop or polled path.
  Its rationale now says the audit narrows the Warn calls and adds the Info and Error lines.
  The acceptance row now reads "No Info-or-above line".

## Disputed

None.

## Deferred

None.
