```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes (`statusStrandSpec`, the `commandDeps` test, the runs-title harness change) for correctness.

**Findings**

1. **LOW** — `internal/loomcli/bootstrap_test.go`, `TestStatusStrandSpec_TitlesTheWindowWithTheRunsHandleAndRecipe`, with `internal/loomcli/sharedbootstrap.go:301`. The test does not pin the defect F4 named.
   - F4 was that reverting `ensureStatusStrand` to pass `c.recipe` passes every test.
   - The test calls `c.statusStrandSpec` directly. Putting `statusStrandAddSpec(statusStrandCmd(...), c.recipe)` back inline at `addSpec := c.statusStrandSpec(exe)` would still pass every test, since the helper would be left uncalled.
   - Reed is the concrete `*reedengine.Engine`, so no fake can capture the spec. The pin that exists covers the helper's content, not the call from `ensureStatusStrand`.
   - The fixer should say in the report that this is a fix with its own change and name the remaining gap. The alternative is a scan or other test that fails when `ensureStatusStrand` stops calling `statusStrandSpec`.

**Traced with no defect**

- **`statusStrandSpec`:** it is the old inline expression moved into a method. `ensureStatusStrand` is its only caller, so no caller changed. `exe` and `c.location` behave as before.
- **Helper test, defect in place:** with `c.recipe` as the title it would produce `"darn"` against the wanted `"T4: darn"` and fail. A wrong command would fail the `Cmd` check.
- **`TestCommandDeps_TitleOpensTheSlugWithTheTasksHandle`:**
  - `commandDeps` uses neither the receiver nor `locateTask` at construction, so `&battenCLI{}` and a nil `locateTask` are safe.
  - `location.WorktreePath()` is evaluated eagerly, but that works for a `locationkit.Location`.
  - The location is `"main"` and the slug is `"run"`, so the test separates slug-from-argument from `WorktreeName`.
  - The test fails on a wrong slug, a wrong board dir or a nil `Title`.
  - The three rows each exercise a distinct outcome (handle found, other slug, no board).
- **`harnessRunsWindowTitle`:** the only other `"beam: prime"` occurrences in `orchcli` tests are `TestRunsWindowTitleOf` and the integration test, which call the real funcs. Nothing else depends on the old stub value. `TestStart_EnsuresTheRunsStrandBeforeTheOrch` now fails if `ensureRunsStrand` stops using `c.runsWindowTitle()` for the spec, add or replace.
- **Doc edits:** the `doc.go` and `cli.go` text matches the code. The keep path in both `ensureStatusStrand` and `ensureRunsStrand` returns without comparing titles.

**Tool calls:** none made. My context already held every changed file and test.
