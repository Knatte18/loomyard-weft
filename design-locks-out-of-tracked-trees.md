# Lock files live in .lym, never beside a tracked file in _lym or Bolt

`locks-out-of-tracked-trees` · Task · bug, fabric, loom · running · PR-Review

## Seen (operator and hub orch, 2026-11-04)

Lock files live beside the files they guard inside tracked trees, kept out of git only by `*.lock` and `*.swaplock` patterns in `.git/info/exclude`.
Running task wefts hold `_lym/reviews/code/generation-record.json.lock`, `settle-record.json.lock` and `settle-record.patch.lock`; the Bolt checkout holds `board.lock`, `board.json.swaplock`, `board.push.lock` and `.gitrepo-push.lock`.
The pattern is general: many state files take their lock as `path + ".lock"` (`state.ReadJSON(path, path+".lock")` across `fabricengine`, `shuttleengine`, `reedengine`, `adoptcli`, the review-record writers in `shedadapters` and `sheddriver`, and others).

The operator's rule: `_lym/` is what git tracks and `.lym/` is machine-local, so a lock file, being machine-local, never lives under `_lym/` or any other tracked tree.

## Idea

- One helper gives the lock path of a tracked file: under the checkout's `.lym/locks/`, mirroring the file's path, never beside it.
- Every `path + ".lock"` and `.swaplock` site whose file sits in a tracked tree (`_lym/`, Bolt) moves to that helper.
- The `*.lock` and `*.swaplock` excludes then go, so a stray lock in a tracked tree shows in `git status` instead of hiding.
- A check (test or lint) refuses a lock path built beside a file under `_lym/` or Bolt.
