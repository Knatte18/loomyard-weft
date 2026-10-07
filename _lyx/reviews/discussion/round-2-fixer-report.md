# Round 2 fixer report

Every finding is fixed; nothing is deferred.
`lyx loom validate-discussion` exits clean on the result.
Edits are in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
Round 1's fixes were checked and none regressed.

## Changes

- F1 (MEDIUM): D3 no longer says `Fabric` holds no location.
  It states that `PushPairAnchored` and the `sync` verb decide from the `Location` they hold, and `Fabric.Commit` from the one it already resolves from its warp path.
- F2 (MEDIUM): D3 states the failure path for all three routes: `push` and `sync` return the prime-name error, and `Fabric.Commit` returns it before taking any lock, so no commit lands and no push spawns.
  The acceptance criterion now names `Fabric.Commit`.
- F3 (MEDIUM): D2's origin deletion requires the probe to have run and answered absent; `Add` carries absent, present or not run.
  D2's Bound states that under `SkipGit` or `SkipPush`, where step 11 still pushes, the rollback deletes the local branch only, and an acceptance criterion pins it.
- F4 (MEDIUM): D2 no longer says a refusal is recorded in `rec`; a refusal or failure is logged and returned, and only a deletion that happened is recorded.
- F5 (LOW): the `PATTERN-mutation-record` bullet now says D4's `side=` rename changes a `detail` value inside the `mutations` entries, which the fixed key set does not constrain.
- F6 (LOW): acceptance criteria gain the mixed `remote:` plus bare `(failed)` retry, an exhausted step 11 (refused and logged when nothing landed, deleted when the ref landed), and the skipped-probe rollback.
  The `Commit` route's failure case sits in the prime-name criterion (F2).
- The support log's interview picks for the rollback and the prime push, and ledger rows Q4 and Q7, carry the changed picks.

## Deferred

None.
