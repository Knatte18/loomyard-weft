```yaml
lens: existing-paths
outcome: reported
```

Lens review: existing-paths, on the decision record and the support log.

**Paths traced, per decision**
- D1, D7 and D8 (the scratch current capture): `RunCapture` would break it if reused, and `RunSideCapture` survives. `looparm`'s `driftengine.Active` survives.
- D3 (scope, `partial`): `LoadConfig` survives once `partial` is read as `full`. Board `run_settings` carrying `skip` is not addressed (finding 5).
- D4 (`Drift-Recheck` on the `DriftVerify` engine): the head-equals-`ComparedHead` early exit breaks after an amendment (finding 1). Resume re-calling an entered row survives.
- D6 (seal): `commitOnExit` makes one commit at producer exit, so the seal breaks or is ambiguous (finding 2). Parent-Align's warp merge survives, because the check compares against the baseline head.
- D11 (`amend`): `runVerb` resolves the run from cwd, so a parent orch cannot reach the verb as written (finding 3).
- D13 (`Verdict`): both callers, `landingdeps.go:244` and `wiring.go` `DriftVerdict`, take `Paths`. They survive if `Verdict` reads `judged.json` through `Paths`.
- D2 bound against `run_settings`: breaks (finding 4).
- D14 (removed verbs): comments in `darn-recipe.yaml` name `lym drift` verbs, so those comments break (finding 6). The orch role has no drift text, so it survives.

**Findings**

1. **MEDIUM — D4 and D11: `Drift-Recheck`'s early exit skips a re-check after an amendment.**
   - D4 makes `Drift-Recheck` Done at once "when the warp head equals the head Drift-Verify last compared".
   - Today `scope.json` records `ComparedHead` together with `ComparePassed` on every compare, failing ones included (`verify.go:323`). `recheck.go:84` exits on `head == scope.ComparedHead` without reading `ComparePassed`.
   - Under the shared engine, a Stuck `Drift-Recheck` compare records its own head. A parent then runs `lym drift amend`, and `lym loom resume` re-calls the row with no new commit. The row is Done at once, so the amended cells are never re-checked or re-judged, and the run lands past a failed check.
   - Fix: the early exit applies only after a passing compare at that head with no amendment since.

2. **MEDIUM — D6: the seal needs two commits, but drift producers commit once, at exit.**
   - D6 commits `expected.yaml`, reads that weft commit's SHA and trailer, then records the seal in `scope.json`.
   - Every drift producer commits only through `commitOnExit`, at exit (`deps.go:63`, called from `await.go`, `baseline.go`, `verify.go` and `recheck.go`).
   - The record names no mid-producer commit and no way to commit the seal record itself. A crash between the two commits leaves a committed `expected.yaml` with no seal. A re-entered `Drift-Expect` must then reseal or refuse, and the record does not say which.
   - Fix: state the two-commit sequence and what re-entry does when an `expected.yaml` commit exists without a seal.

3. **MEDIUM — D11: the parent orch cannot reach `lym drift amend`.**
   - Every `lym drift` verb resolves the run from the command's cwd (`driftcli/cli.go:55-56`, `runVerb`). A parent orch runs in its own worktree.
   - `lym loom decision add` takes a `<slug>` and checks the caller's strand (parent orch, arbiter or sampler).
   - D11 names neither a slug argument nor how the caller's identity is checked. "Admitted for the run's parent orch" is therefore unreachable, or unchecked if the parent changes directory.
   - Fix: `amend` takes the run's slug, and checks the caller as `decision add` does.

4. **MEDIUM — D2 and D3 bounds: a task's `run_settings` can set any drift key.**
   - D2's bound says only the operator chooses `command` mode, "in the hub's `drift.yaml`". D1 implies `key_patterns` is hub config.
   - `internal/configreg/runsettings.go` refuses only modules or keys declared retired; it has no allowlist. A task's `run_settings` can therefore set `drift.diff: command`, `drift.key_patterns: []` or `drift.scope: off`, and so switch off containment by code.
   - The non-executable keys are open to the parent through `run_settings`, while `capture` and `compare` are executable and closed.
   - Fix: restate the bound to include the parent's `run_settings`, or refuse `diff` and `key_patterns` there.

5. **LOW — D15: board entries' `run_settings` are not migrated.**
   - D15 relies on `lym config reconcile` and a hub-base reinterpretation. A board entry's stored `run_settings` (`{"drift":{"scope":"partial","skip":[...]}}`) are seeded at Seed-Child, where only a warning fires.
   - Declaring `skip` and `verify_max_runs` as retired keys (`refuseRetiredKeys`, `runsettings.go:41`, `:97`) would refuse such a seed with a way forward. The record should pick that or the silent warning.

6. **LOW — D14: recipe comments name the removed verbs.**
   - The grep of `lym drift accept|withdraw|waive|scope|compare` also hits `contracts/recipes/darn-recipe.yaml`, whose comments, such as the `waive` line near l.53, would go stale. D14 lists stencils and code but not the recipe comments.

7. **LOW — D7 and D8: the current capture must not reuse the main-capture path.**
   - "Under the capture lock … as today's capture does": the main path, `RunCapture`, rewrites `capture.json` and moves a non-empty baseline to a numbered backup. Drift-Await's counted capture reads that state.
   - D7 should say the current capture writes no `capture.json` and never touches the baseline directory. It should also say whether a held lock makes a concurrent `lym drift check` wait or refuse; for example, the writer's gate entry and a Webster fork's `check` running at once.

**Tool calls**
- grep of `internal/configreg/runsettings.go`, `boardcli`, and repo-wide references to the removed `lym drift` verbs: my context did not show whether `run_settings` restricts drift keys or where removed verbs are named.
- grep of `commitOnExit`, `driftcli`'s caller resolution, `contracts/orchrole` and readers of `baseline.md`: my context lacked the commit timing, how verbs find the run, and the outside readers.
- grep of loomcli drift users, `runsettings.go` refusal rules, and `ComparedHead` and `BaselineHead` in `recheck.go` and `verify.go`: needed to trace the resume and early-exit path and the `run_settings` key rules.
