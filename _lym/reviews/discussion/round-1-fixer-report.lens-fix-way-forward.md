```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, step and criterion the fix added can be carried out and makes progress.

Ways forward the fix introduced:
1. D1 adds refusals of `--stop-on-fail` and `--dir`/`-C`. The way forward is "`lym gate test --map` with targets or a scope flag". It can be carried out and makes progress.
2. D1 says `--main-packages` applies with `--card`. That is a behaviour, not a refusal, and it is reachable.
3. D1 and D4 refuse a hub whose tier sets `memory_mb` below `redundancy_test_memory_mb`. The way forward is to raise `memory_mb` or lower `redundancy_test_memory_mb` with `lym config gate`. It can be carried out:
   - `tiers` is a map (`internal/gateslot/template.yaml:18`).
   - `lym config gate --set` takes dotted map keys, as `tier_parallel.tier0=8` does in `configcli_integration_test.go:215`.
   - Either change makes the quotient at least 1, so the map call goes ahead.
4. D9 now has every map-rebuild refusal of `lym gate redundant` name `lym gate test --map &lt;package&gt;`. That includes "built from other Go files than the worktree holds". See finding 1.
5. New criteria for D2 (deleted last test), D4 (refusal) and D9 (failed-build-only map) can all be reached from the states they name.

Findings:

- MEDIUM. Changed text: D9, "Every refusal of `lym gate redundant` that names a map rebuild names `lym gate test --map &lt;package&gt;` from the worktree as its way forward: … or built from other Go files than the worktree holds."
  Summary: when the mismatch comes from uncommitted edits in the worktree, the way forward leads to D6's dirty-package refusal and never to a map.
  Evidence:
  - `internal/gatecli/redundant.go:224-228` hashes the working tree's Go files (`goFileHashes(workingTree{root: location.WorktreePath()}, …)`) and refuses on any difference from the map. A modified, uncommitted Go file in the package triggers it.
  - D6 has the map run write no map for a package with a modified or untracked file it owns, and list the package as unmeasured.
  - So the named next step cannot make progress until the edits are committed or reverted, and the refusal does not say so.
  - The fix newly lists this refusal under D9. The way forward needs to say that the edits are committed or reverted first, or the criterion has to accept a refusal that names that step.

Tool calls beyond context:
- Read `internal/gatecli/redundant.go:175-230` and grepped `internal/configcli/doc.go`. My context held only lines 162-174 and 228 of `redundant.go`, so I could not tell whether the stale-files check reads the working tree or HEAD. I also needed to know whether `lym config gate --set` can reach a tier key.
- Grepped `memory_mb` and `tiers` in `internal/gateslot/template.yaml` and the `memory_mb` refusal in `internal/gateslot/tiers.go`. My context did not show whether `tiers` is a map that a dotted `--set` key can address.
