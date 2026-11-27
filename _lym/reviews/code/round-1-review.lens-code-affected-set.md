```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I traced everything that calls, reads, asserts or describes it, and report each member the diff leaves wrong.

**Per change, the set found:**

1. **`needsPrebuiltLym` / `invocationsUseLymBinary` gate both prebuild call sites** (`internal/gatecli/tierrun.go:558`, `internal/gatecli/copies.go:65`).
   - Callers: `runInvocations` and `runTierCopies`. Covered.
   - Moving `run.tier.PrebuiltLym = false` out of the `if` in `copies.go:81` leaves the copies unchanged: with the flag false, `needsPrebuiltLym` returns false. Covered.
   - Env: `gateEnv` strips any inherited `LYM_PREBUILT_LYM` (`internal/gatecli/run.go:912`), so a skipped run exports none. Covered.
   - Tests: `TestRunTierCopies_BuildsThePrebuiltLymOnce` (`copies_test.go:240`) has an invocation with no target, so it still builds. Covered. The scenario prebuild subtests (`test_integration_test.go:1957-2090`) target `./pkg`, which is absent in the fixture, so they still build. Covered.
   - Docs:
     - `internal/gatecli/doc.go:106-112`: updated.
     - `prebuild.go` file header: updated.
     - `tierrun.go:213/222` comments: updated.
     - `copies.go:62`: updated.
     - `internal/gateslot/tiers.go:48`: **not updated** (F1).
     - `internal/gateslot/pool.go:29-30`: **not updated** (F2).
     - `docs/benchmarks/running-tests.md:90`: **not updated** (F3).
     - `internal/gateslot/doc.go:51-52` ("keeps the `lym` it built"): still true. Covered.
     - `internal/gateslot/template.yaml:10`: names the key only. Covered.
2. **New `impactset.TestsUseLymBinary`, which reuses `lymbinDir`** (`internal/impactset/graph.go:13`).
   - The `impactset` package doc and the `tiertests.go` header are updated. Covered.
   - The importers the change description counts (26 test files in 11 directories) match a direct-import grep. Covered.
   - The Testkit Invariant's `kitDeniedImports` holds `lymbinImport` (`internal/testkit/enforcement_test.go:27`), so the claim that no other kit wraps the kit holds. Covered.
3. **The rule doc `pattern/PATTERN-test-speed.md:75`.** Updated, but it states only some of the conditions that build (F4).
4. **The `PATTERN.md` entry `PATTERN-test-speed`.** Its summary says nothing about the prebuild condition, so nothing is forced. Covered. The directive declares no PATTERN change, so the "declared changes landed" check raises nothing.
5. **Commit messages** for `bffe6e2b2`, `ebe7b009c` and `1228219e9`. They agree with the code: no target, a `...` pattern, an import path or the kit's own directory each builds. No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none is required. Covered.
6. **Stencils.** None hold `prebuilt_lym` or describe the prebuild. Nothing is affected.

**Findings:**

- **F1 — LOW, consistency, `internal/gateslot/tiers.go:48`.** The `PrebuiltLym` field doc still says the tier always prebuilds lym ("makes a run of the tier prebuild lym from the worktree first"). It now prebuilds only when a test of its targets may use the binary. Evidence: `internal/gateslot/tiers.go:48` against `internal/gatecli/prebuild.go:32-41`.
- **F2 — LOW, consistency, `internal/gateslot/pool.go:30`.** The `PrebuiltLymEnv` doc says "A tagged gate run builds it once for all packages". A tagged run whose targets' tests don't import `lymbin` now builds nothing and exports nothing. Evidence: `internal/gateslot/pool.go:29-30`.
- **F3 — LOW, consistency, `docs/benchmarks/running-tests.md:90`.** It says the harness always builds lym once for a tagged tier, "as `lym gate test --tags` does". After the diff, `lym gate test` builds only when a targeted package's tests use the binary, so the comparison no longer holds. Evidence: `docs/benchmarks/running-tests.md:90`.
- **F4 — LOW, consistency, `pattern/PATTERN-test-speed.md:75`.** The rule doc says the run builds only for a test file importing the kit or a pattern target. The code also builds when a target is no directory path (an import path), when an invocation has no target, or when the kit's own directory is targeted (`internal/gatecli/doc.go:109-110`). So the "only" sentence describes a narrower rule than the code runs. Evidence: `pattern/PATTERN-test-speed.md:75` against `internal/gatecli/prebuild.go:80-96`.

Tool calls:
1. A grep of every `.md`, `.go`, `.yaml` and `.tmpl` mention of prebuilt/prebuild outside `_test.go`. My context held only a subset of the docs and comments that describe the prebuild.
2. `pattern/PATTERN-test-speed.md:68-82`, plus a grep of docs for `prebuilt_lym` and of the config and stencil directories. My context held only the diff hunk of the background file and none of the stencil or config docs.
