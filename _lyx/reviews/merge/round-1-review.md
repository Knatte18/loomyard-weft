---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/hubreconcile/ensure_integration_test.go:1295
    summary: no test replays the task's strip commit, which adds the base-sparse marker, over a moved upstream through the parent's pull-first replay
    fix: add a row to the moved-upstream test (or a `boardBehind` row combined with `unmigratedBoard` in `runStaleBuildReconcile`) that starts from a board holding the pre-change full base files and no marker, moves upstream on another path, runs Ensure, and asserts the board commit holds the stripped base files and the marker over the moved upstream and that the stamp is written.
    origin: lens:code-tests
---

### [LOW] Strip commit with a new marker file is never replayed over a moved upstream

**Location:** `internal/hubreconcile/ensure_integration_test.go:1295`
**Issue:** The parent's pull-first replay and the task's one-time base strip meet in `reconcileHubWide`, where the strip commit adds the marker file and rewrites base files. The moved-upstream tests start from `NewHub`, whose board already carries the marker, so the replay never sees a commit that adds the marker. No test combines an unmigrated board with a board behind its upstream.

## Rejected

- Stale `walkHub` doc comment ("skipped or left unpushed") and the Info line in `ensure` ("not committed and pushed") after the push-skip was removed: the parent's change alone made them stale, so it is not the interaction.
- No test for the `spawnBoardSync` failure Warn branch: the branch is the parent's own code and not touched by the task.
- Config walk running before the origin reconcile in `orchcli` and `battencli`: the order predates both sides, and the parent's docs scope its ordering claim to the strand decision.
- `PATTERN.md` carrying the short `PATTERN-sparse-hub-base` text instead of the declared text: it lives in the task's own range, which the Webster review covered.
- Way-forward wording of `WorktreeError.Board` against the parent's `reconcileFailureWayForward`: they cover different failures and neither names the withdrawn `git pull --rebase`.
