```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff's changed functions and branches. Each entry gives the case I ran and whether it was correct.

**List**

1. `pruneSparse`, input with no unknown keys (`removed` empty): the early return gives `pruned`, `removed`, `kept` and nil `preserved`. Correct. A rerun by the older binary still sees the unknown keys in the first `Prune`'s `removed`, so `preserved` is recomputed each time.
2. `pruneSparse`, `current = "newer_key: 1\npublish_verify: x\n"`, no `PruneUnknown`:
   - The first `Prune` removes both keys.
   - `DropRetired` removes `publish_verify`.
   - The second `Prune` gives `preserved = [newer_key]`.
   - `removed` becomes `[publish_verify]`, and the returned bytes are `withoutRetired`.
   - Correct.
3. `pruneSparse`, only unknown keys, so `removed` is empty after `DeleteFunc`: `DropRetired` returns the input byte for byte (`layers.go:224`). `changed` stays false in `ReconcileHubWideAt` (`configsync.go:1019`) and in `reconcileSparseFile` (`:605`), so nothing is rewritten and no pending change is reported. Correct.
4. `pruneSparse`, `PruneUnknown` true: returns the first `Prune`'s result untouched. Correct.
5. `pruneSparse`, `Prune` or `DropRetired` fails: errors propagate. In `ReconcileHubWideAt` they pass through the `reconcileErr` / `FileError` wrapping, and in `reconcileSparseFile` through `fail("reconcile", err)`. Correct.
6. `pruneSparse`, retired key nested under an unknown parent (`x.old` retired, `x` not in the template, file `x: {old: 1}`):
   - `DropRetired` leaves `x: {}`.
   - The second `Prune` reports the empty mapping as unit `x` in `preserved`.
   - `removed` keeps `x.old`, and the file keeps `x: {}`.
   - Finding F1 below.
7. `pruneSparse`, a retired key emptying a template mapping: `DropRetired` leaves `a: {}`, where `Prune` would have dropped the empty mapping (`yamlengine/layers.go:428`). It is not reported in `preserved`. The same F1 mechanism applies.
8. `pruneSparse`, output encoding: when `preserved` is non-empty, the bytes come from `DropRetired`, which uses `SetIndent(2)`. The `Prune` path uses `yaml.Marshal`, whose default indent is 4. The two paths can re-indent the file differently. This only matters on a rewrite that was already happening, so I did not raise it as a finding.
9. `ReconcileHubWideAt`, `changed` and the marker or build-stamp write when every removal is preserved: `changed` is false, nothing is written, and `Preserved` is set on the `Result` in dry runs and applied runs alike (`:1054`). Correct.
10. `ReconcileAll`, local file with `pruneUnknown=true` (`:452`): behaves as before the diff. Correct.
11. `ReconcileAll`, worktree file with `opts.PruneUnknown`: passed through. Correct.
12. `reconcileBaseWorktreeFile` (standalone) ignores `PruneUnknown` and still removes unknown keys through `yamlengine.Reconcile` (`:546`). The only non-test `ReconcileAll` callers that can pass an empty board dir are manual: `configcli` runReconcile, which sets `PruneUnknown`. `clone.go:122` and `walk.go:285` always pass a board dir. Not a defect.
13. `sparseSeed` (`:755`) still calls `yamlengine.Prune` and drops unknown keys from a prime or legacy seed of an absent file. See F2.
14. `DropRetired` export and the `configcli` and `hubwide.go` `PruneUnknown: true` sites: correct. The `hubwide.go:184` opts are reused for the dry run, the applied run and `applyOpts`.

**Findings**

- **F1 — LOW, design**, `internal/configsync/configsync.go:641-653`. When a retired key sits under a parent the template lacks, or leaves a template mapping empty, `DropRetired` leaves an empty mapping (`x: {}`).
  - The first `Prune` would have dropped that empty mapping, but the preserved branch returns `withoutRetired` unpruned.
  - For an unknown parent, `preserved` also reports the stale empty mapping as a preserved key.
  - The leftover mapping is harmless, and only retired keys nested under such a parent trigger it.
- **F2 — LOW, design**, `internal/configsync/configsync.go:755`, called at `:984` and `:989`. `sparseSeed` still runs `yamlengine.Prune` without preservation.
  - A newer binary's key held in the prime worktree's copy is dropped when an absent hub default is seeded from it.
  - `PATTERN-config-layers` (new section) says every reconcile other than `lym config reconcile` keeps and reports such keys.
  - The prime's own file is untouched and this path only seeds an absent file. The rule text and the code still disagree.

**Tool calls**
1. Read `internal/yamlengine/layers.go:430-510`. My context had `Prune` only to line 431, and I needed the rest of `pruneMapping` and its empty-mapping handling.
2. `grep -rn "ReconcileAll("` over non-test Go files. I needed to check whether any automatic caller reaches the standalone base path.
