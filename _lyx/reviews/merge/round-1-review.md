---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/configcli/reconcile_integration_test.go:52
    summary: no test covers the interaction of a reconcile --layer default --apply with a rules file left uncommitted in Bolt
    fix: in the existing default-layer apply test in internal/configcli/reconcile_integration_test.go, seed an untracked _lyx/orch/rules.md in the board repo before the apply, then assert the new commit's file list holds only _lyx/config/ paths and that rules.md stays untracked with unchanged content; add this as an assertion in that test, not a new test
    origin: lens:code-tests
---

### [LOW] No test pins that reconcile leaves an uncommitted hub-wide rules file alone

**Location:** `internal/configcli/reconcile_integration_test.go:52`, `internal/orchcli/rules_integration_test.go:19`
**Issue:** `reconcileDefaultLayer` (parent) commits only `ConfigFileRel` paths under the board write lock, and `lyx orch rules` (task) can leave `_lyx/orch/rules.md` written but uncommitted after a failed commit.
The code is correct by reading: the scoped pathspec cannot stage the rules file, and `snapshotBoardConfig` and `restoreBoardConfig` cover only the config dir.
No test asserts it, so a later switch to a whole-repo commit would sweep the rules file into a config commit unnoticed.

## Rejected

- Shared commit-on-Bolt behavior covered twice: the `orchcli` rows use a fake Bolt and the `configcli` tests use a different seam, and the scoped pathspec is pinned once in `fabricengine`; no redundancy.
- Docs describing Bolt's `_lyx/` (`internal/configcli/doc.go`, `internal/configengine/doc.go`, `docs/shared-libs/configengine.md`): they speak only of `_lyx/config/`, and the merged `docs/overview.md` orch and config lines are each accurate.
- Reed and creel changes against `orchengine`: only unexported internals changed, no exported signature the orch code calls moved.
- Board write lock sharing: both verbs take the same lock, with disjoint scoped paths and no nested acquisition.
