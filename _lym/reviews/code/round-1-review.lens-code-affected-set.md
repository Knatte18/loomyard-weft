```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found:**
- **New gate keys** (`test_map_max_age_days`, `redundancy_build_timeout_min`, `redundancy_test_timeout_sec`): covered.
  - `internal/gateslot/config.go` and `template.yaml` carry the keys, with the D5 defaults 7, 10 and 120 as cross-pair.
  - `config_test.go` has the `templateDefaults` update and zero-refusal rows, and `doc.go` documents them.
  - Other readers of `fixer_timeout_min` (`configreg_test.go`, the `hubreconcile` test) concern the landing config and are unaffected.
  - `baseKeys` is `"slots: 2\n"`, so the template layer supplies the defaults and the new "at least 1" refusal does not break existing fixtures.
  - No existing key changes meaning, so no reinterpretation is needed.
- **`lym gate testmap` verb**: covered.
  - `cli.go` registers it under the existing gate subtree.
  - `gatecli/doc.go`, `gateslot/doc.go`, `docs/overview.md` (module table and gate verb list) and `testcover/doc.go` are updated.
  - The `shellguard` steer text names only `gate test` and `card-verify`, so it needs no change.
- **`gateEnv` extraction and `prebuildLym` group kill**: covered.
  - The only other caller of the env block is `runTest`, which now calls `gateEnv`.
  - The `prebuildLym` doc comment states the new group-kill behaviour.
- **New exports** (`fabricengine.CodeDirAt`, `CodeFileAt`, `hubgeom.IsPrime`, `PrimeWorktreePath`, `lymdirs.TestMapExitFileName`, `testcover.Dir`, `ExitRecord`): covered. Each has doc comments, and the fabric methods go through `gitrepo`'s existing `FilesInDirAtRevision` and `FileAtRevision`.
- **PATTERN entries**: `PATTERN-gate-slot` (non-tier holder listed in the `gateslot` doc), `PATTERN-fabric-boundary`, `PATTERN-lym-files` (path named by `lymdirs`, lock via `lock.TrackedFileLockPath`), `PATTERN-child-processes` (spawns logged) and `PATTERN-refusal-way-forward` (not bound to `gatecli`) are all satisfied.
- **Declared PATTERN changes**: none are declared, and none are needed.
- **Commit messages**: none carries a `Pattern-Departure:` trailer, and none is needed.
- **Stencils**: none changed.

**Finding:**
- **LOW, consistency.** `internal/gatecli/doc.go` (the Coverage maps section, line "the wait for its exit and output lasts at most the ten-second pipe wait") pins a perishable value.
  - The grace is passed in as `pipeWaitDelay` (`cli.go:51`, `cli.go:171`), so a change to that value falsifies the doc with nothing to catch it.
  - Fix: write "at most the pipe wait the gate already uses" without the number.
  - I did not confirm the current value, because the grep did not reach where it is set.

**Tool calls (3):**
1. Grep for `fixer_timeout_min` and `card-verify` across the repo, to find other readers of the key and other lists of gate verbs. My context had no repo-wide search.
2. Grep of `configreg_test.go`, `hubreconcile/ensure_integration_test.go`, `docs/overview.md`, `shellguard/rows.go` and the `baseKeys` definition, to check whether those hits depend on the changed keys or verbs. My context lacked those lines.
3. Grep for the `pipeWaitDelay` definition, to check the "ten-second" claim. It found only parameter declarations, not the value.
