```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked whether the claims the fix added or rewrote hold against the code at HEAD.

**[MEDIUM] D7's last paragraph and its new criterion: "`gates` is a per-name map, so a hub base `gate.yaml` that sets one of those keys keeps its own unit over the template's"**

The code points the other way, and nothing here shows a per-name entry replacing the template's entry whole when config loads.

- The load path calls `yamlengine.FillMissing(spec.Template, stackDoc, spec.OpenMaps...)` (`internal/configengine/config.go:278`).
- For the gate module, `ConfigOpenMaps` lists only `tiers`, `tier_parallel` and `build` (`internal/gateslot/config.go:99-101`), so `gates` is not one of them.
- `FillMissing` "merges every mapping key the template holds and existing lacks into existing's own document" (`internal/yamlengine/fill.go:14`). It skips only open-map paths, so it goes into `gates.webster` and would add a template `reports` key that the hub's entry lacks.
- `config.go:282-285` drops "a name filled under a per-name map" from the fill log as "the normal case", which confirms that filling under `gates` happens at load.
- The doc of `ConfigPerNameMaps` says entries are "merged name by name" (`config.go:103-104`). The only "replaces ... one entry whole" rule found applies to an overlay write (`configengine/doc.go:155`). `mergePerNameMaps` is part of reconcile (`yamlengine/reconcile.go:256`), not load.

So the new D7 claim, the new reinterpretation built on it, and the Constraints line "The one reinterpretation that ships…" rest on a merge behaviour the code does not appear to have. This is the premise of review finding F1, carried over without checking. Either:
- the record should cite where config load treats a `gates` entry whole, or
- if load fills `reports` from the template anyway, the reinterpretation is redundant. In that case the record should say the template alone reaches this hub, and F1 is disputed with `premise false`.

**[LOW] D7: "A repeat call on a tree its report matches pays only the diff read and the hashes"**

This understates the cost. D8's match runs D7 step 1 again, which reads each changed `_test.go` file's text at the merge base through `fabricengine.CodeFileAt` and hashes each test function. That is a git read per changed test file, on top of the diff read and the package file hashes.

**Claims checked that hold**

- `cardChanges` and `impactset.ChangedPaths` return only paths, and untracked files are included (`impactset/targets.go:102-105`).
- `fabricengine.AnchoredFileAt` reads the weft through `RecordsWorktree` (`trailercommit.go:151-153`).
- `Fabric.MergeBase` reads the warp checkout (`warpforward.go:45-54`).
- `parentMergeBase` reports false for the prime on main (`scope.go:668-677`).
- The merge takes the weft write lock and then runs `MergeStart` in the warp checkout (`merge.go:421-488`).
- `runproc.List` returns only live, running records (`record.go:129-215`), so "in flight while a live `test-map` record exists" can work.
- `fabricengine.AppendExcludePaths` exists as a precedent for an appending reinterpretation.
- The hub reconcile walk applies a reinterpretation once behind a board marker and sends a notice to the orch (`hubreconcile/doc.go`).
- `this hub's _bolt/_lym/config/gate.yaml sets all four with no reports` matches `gate.yaml:46-51`.

**Not verified:** D6's "A dobby stop does not end the child" and "releases its slot and locks as it exits". `internal/dobbyengine` has no `HubOwner` or stop path to check against, and the release mechanism of `internal/lock` was not inspected.

**Tool calls made**
1. `impactset.ChangedPaths` doc, `runproc.List` and liveness, and the per-name map users: to check the diff-route, in-flight and per-name claims.
2. `configengine/config.go:270-300` and the `doc.go` "unit" lines, plus the gate doc lines on `gates`: to check how a `gates` entry merges with the template.
3. The `configengine` layer-merge doc, the `FillMissing` signature, gate `ConfigOpenMaps`, the dobby stop and the `lock` doc: to check the fill path and the D6 child claims.
4. `yamlengine/fill.go:1-21` and `ConfigPerNameMaps`: to check whether `FillMissing` goes into `gates` entries.
5. The template's `gates` lines and the per-name merge use in `yamlengine`: to see whether merging per-name entries whole is limited to reconcile.
