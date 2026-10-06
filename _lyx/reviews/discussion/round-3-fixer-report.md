# Round 3 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

## Fixed

- F1: D7 now states that `checkout` adopts an origin weft whether or not an archive tag covers it, with its bound (no pair created, nothing on origin changes, the archived weft's records come into the worktree).
- F2: D7 now covers the raw-warp and unmanaged reconcile cases together, states that both end as `weft_recreated` with the wiring repair where the raw path ended as a dormant `raw_adopted`, and states the weaker bound for a raw worktree.
- F3: D6 now has an adopted weft that already carries an origin record keep it, with no rewrite or commit; only a weft without a record gets one naming the acting worktree.
- F4: D4 now states that a `QueueNotice` error is a logged warning and that a notice only logged for lack of an orch strand is the accepted bound; A1 and the diff-read-failure acceptance clause are aligned.
- F5: added acceptance clauses for the reconcile unmanaged path, for `checkout` onto an archive-covered origin weft (folded into the existing checkout clause), and for the kept origin record on adopt.

## Deferred

Nothing deferred.
