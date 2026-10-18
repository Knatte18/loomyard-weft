---
round: 1
focus:
  - "internal/configcli/hubwide.go (parent) against internal/orchcli/rules.go (task): both commit files under Bolt's `_lyx/` through `_board`. Settle whether they share one commit/push path or lock, whether `lyx config reconcile --layer default --apply` can stage, rewrite or clobber `_lyx/orch/rules.md`, and whether the two keep the same prime-only and commit-and-push conventions."
  - "Parent's changes to `configengine` layers and `fabricengine.BoardDir` use (internal/configengine/config.go, layers_test.go) against the task's use of `fabricengine.BoardDir` in `internal/orchcli/paths.go`. Settle whether the board-dir resolution the task joins `HubRulesPath` from still means what the parent's layer code means."
  - "docs/overview.md, the one overlapping file: the orch module line (task added the `rules` verb) and the config line (parent added `reconcile --layer default`) merged textually. Settle whether the merged text is accurate for both verbs, and whether `docs/shared-libs/configengine.md` or `internal/configcli/doc.go` (parent) still describe `_lyx/orch/` on Bolt correctly now that the orch rules file lives there."
  - "Parent's `internal/reedengine/probe.go` and `internal/creelengine` changes against `internal/orchengine` (task: `prompt.go`, `watcher.go`, `state.go`, `Paths.PrimeRoot` in `isRequestedNote`). Settle whether the orch watcher or strand code still calls the reed and creel symbols and contracts in the shape the parent left them."
  - "Test economy across the merge: whether the parent's new `configcli` hub-wide tests and the task's `orchcli` rules tests cover any shared commit-on-Bolt behavior twice, and whether the interaction (reconcile touching Bolt `_lyx/` while a rules file exists there) has any test."
---

Seeded from the merge note: the only overlapping file is `docs/overview.md`, no packages overlap and no pattern files changed.
The interaction to look for is semantic, around Bolt's `_lyx/` directory, which the parent's hub-wide config reconcile and the task's hub-wide orch rules file both use.
