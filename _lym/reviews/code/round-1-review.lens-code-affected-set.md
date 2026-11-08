```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. For each change, I list the set I found and whether it is covered.

**Per change**

1. `HolderEnv` GOFLAGS now opens with `-count=1`.
   - Production callers: `internal/driftengine/steprun.go:134`, `internal/cardverify/runner.go:95`, `internal/planglyph/typesload.go:112` and `cmd/testtiming/resources.go:141`. All inherit the flag, so they are covered.
   - Tests that assert the value are updated: `internal/gateslot/pool_test.go` and `internal/planglyph/glyphchain_integration_test.go:660`.
   - Tests that assert only a substring or suffix still hold: `internal/cardverify/runner_test.go:143` (`-p=6 -trimpath`) and `cmd/testtiming/takegateslot_integration_test.go:43,65` (suffix checks).
   - Not covered: the `loadEnv` doc comment (F1 below).
2. `WithTestCacheOff` is applied in `runTier` (`internal/gatecli/tierrun.go:90`).
   - `runTier` is called only from `internal/gatecli/run.go:491` and `:530`, so every tier invocation is covered, including the guard step and the no-slot and inherited-slot paths.
   - `trimpathLine` still reads the un-flagged `tierEnv` (`tierrun.go:85`), as the docs say.
   - The `go test`-bearing tier commands are all covered.
   - Not covered: the `TestCacheOffFlag` doc over-claims its reach (F2 below).
3. Docs.
   - `internal/gatecli/doc.go` (new "Test result cache" section and the trimpath wording) and `internal/gateslot/doc.go` (the two lines about the build cache and the `HolderEnv` addition) are updated and consistent with the code.
   - `docs/benchmarks/*` and `docs/research/*` mentions of the test cache are measurement reports, which the rubric exempts.
   - `pattern/PATTERN-test-speed.md` and `pattern/PATTERN-gate-slot.md` have no text about `GOFLAGS` or `-count`, so nothing there is left stale.
4. Declared PATTERN changes: none declared, so nothing to check.
5. Commit messages: no `Pattern-Departure:` trailer and no difference that needs one.

**Findings**

- **F1 — LOW, consistency.** Location: `internal/planglyph/typesload.go:102`. The `loadEnv` doc comment says `HolderEnv` appends "`-trimpath` and `-p` ... after `-mod=readonly`". `HolderEnv` now also puts `-count=1` before `-mod=readonly`, and the test at `glyphchain_integration_test.go:660` asserts it. Evidence: `pool.go:479-486` and the comment at `typesload.go:102`. Fix: name `-count=1` first, before `-mod=readonly`.
- **F2 — LOW, consistency.** Location: `internal/gateslot/pool.go:432`. The `TestCacheOffFlag` doc says the gate puts it in the GOFLAGS "of every command it runs". Three commands run without it:
  - the build step: `internal/gatecli/build.go:48`, which uses `renderBuildEnv`
  - `go list`: `internal/gatecli/green.go:509-511`
  - the `lym` prebuild: `internal/gatecli/prebuild.go:61`

  Of these, only a hub's configured build-step command could ever run `go test`. `go list` and `go build` ignore the flag, as the summary says. Fix: reword to "every tier command it runs" or "every tier command and non-tier slot holder's child". The commit message ("every Go command") and the `tierrun.go:66` comment ("Every command's GOFLAGS") need the same scoping.

**Tool calls (3):**
1. A grep for other `GOFLAGS` assertions in tests, docs that mention the test cache, and `typesload.go` lines 95-115. My context lacked all of these.
2. A grep for the `runTier`, `runInvocations`, `exec.CommandContext` and `renderTierEnv` call sites in `internal/gatecli`, plus `-count` mentions in the two `pattern/` files. This checked for tier paths that skip `WithTestCacheOff`.
3. Reads of `green.go:480-520` and `build.go:35-60` to see what the `go list` and build-step commands run.
