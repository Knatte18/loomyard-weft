---
round: 3
focus:
  - "Hub-wide write paths of `lyx config darn`: the `--set` path (`setHubWide` in `internal/configcli/hubwide.go`) checks the result with darn's own loader before and after and rolls back, while the editor path (`editHubWide` through `copyStagedToHub`) validates only through `configengine.Load`. Settle whether the decision record makes both paths refuse a save that leaves `tree_scope_verify` set with an emptied `tree_scope`, or holding duplicate-after-trim keys, and whether D1's rollback claim is true for each path."
  - "Tree keys and diff paths: the changed-path read behind `ChangedPaths` in `internal/fabricengine/parenttips.go` uses `git diff --name-only` without `-z`, and git C-quotes a path outside core.quotePath's ASCII set. Settle whether the record states that paths are reported verbatim, and whether D1's refusal list and trailing-slash trim decide a key with an empty segment such as `tools//x` and a key such as `tools//`."
  - "D1's refusal list and its on and off sequences: settle whether each refusal states its own condition, so the empty `tree_scope_verify` the template ships and `lyx config darn --set tree_scope_verify=` stay accepted, and whether the order for removing the last tree while `tree_scope_verify` is set is stated."
  - "D5's `--guards` failure handling: `FindGuardTests` in `internal/impactset/guards.go` returns `GuardScanError` for its content kinds and a plain wrapped error when a package directory cannot be read. Settle whether the record states the verb's behaviour and the message's content for each kind, before any gate slot is taken."
---

Round 2 left one gating-class finding on a reopened key, the tree-key gaps, and one new MEDIUM on the editor path.
The entries above target the text the fixer adds for them, which no fresh reviewer has seen.
