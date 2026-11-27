```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the goal points of the board entry.

**Decisions** (the board entry's goal points; the record has no `Added after Discussion` entries):
- **G1:** build `lym` only when a test file of a targeted package imports `internal/testkit/lymbin`.
- **G2:** first check every route by which a test reaches the prebuilt binary and count each one, so no test loses the binary it needs.
- **G3:** a call over the whole set still builds it.
- **G4:** the hub's tier1 command change stays out of this task.

**Changes and what they trace to:**
- **C1** traces to G1: `impactset.TestsUseLymBinary` in `internal/impactset/tiertests.go:35-64`, with its row table `TestTestsUseLymBinary` in `tiertests_test.go`. It reads a directory's `_test.go` imports.
  - The kit's own directory also counts, which traces to G2: `lymbin`'s own in-package tests are a route to the binary.
- **C2** traces to G1 and G3: `needsPrebuiltLym`, `invocationsUseLymBinary` and `isDirectoryTarget` in `internal/gatecli/prebuild.go:66-108`, wired in at `tierrun.go:558` and `copies.go:65`.
  - No target, a `...` pattern or a non-directory target builds, which is G3.
  - The unconditional `run.tier.PrebuiltLym = false` at `copies.go:81` is forced by moving the condition into `needsPrebuiltLym`; it changes no behaviour.
- **C3** is tests, forced by C2: `TestInvocationsUseLymBinary` in `prebuild_test.go` and the new no-build subtest in `test_integration_test.go:2096-2115`.
- **C4** is docs and comments C1 and C2 made stale, so it is forced:
  - the `gatecli` `doc.go`, the `prebuild.go` file header, and the comments in `tierrun.go` and `copies.go`;
  - the `impactset` `doc.go` and the `tiertests.go` header;
  - `pattern/PATTERN-test-speed.md:75`.
- **C5** is commit `1228219e9`, which re-breaks C1 to C4's own comment lines. It is forced and adds no scope.

**Decisions and the changes that build them:**
- G1 is built by C1, C2 and C3.
- G2 is built by C1's kit-directory rule and C2's "uncertain cases build" rule. The route count lives in the change description and the `doc.go` paragraph on the Testkit Invariant. I checked the description's "26 test files in 11 package directories" against a grep of real import lines in `internal` and `cmd`, and it holds.
- G3 is built by C2's no-target, `...` and non-directory branches, but only partly. See F1.
- G4 holds: the diff touches no hub config (`git diff --stat` lists only `internal/gatecli`, `internal/impactset` and `pattern`).

No change lacks a decision behind it, so there is no unrequested-scope finding.

**Findings**

**F1 — MEDIUM, design (borders on scope: G3 partly unbuilt).** Location: `internal/gatecli/prebuild.go:86-102`.
- **Summary:** a call with explicit targets on a prebuilding tier whose command holds neither `{targets}` nor `{test_targets}` runs the whole set without the prebuild.
  - The decision reads the caller's targets even though the command ignores them.
  - The command then runs every package's tests with no `LYM_PREBUILT_LYM`, so each test binary that imports `lymbin` compiles `lym` on its own.
  - No test loses the binary, but the whole-set run does not build it, which goes against G3.
- **Evidence:**
  - `internal/gatecli/tiers.go:298` returns `targets: request.targets` with `ignoresTargets` true when `!gateslot.HoldsTargets(tier.Command)`. `tiers.go:154-156` collects such a tier in `wholeTiers` as running whole.
  - `invocationsUseLymBinary` at `prebuild.go:86-102` never looks at `run.tier.Command`.
  - The scenario fixture has a tier of this shape: `tier2: standIn+" whole"` at `test_integration_test.go:642`.
  - `lymbin.buildOnce` compiles when the variable is unset (`internal/testkit/lymbin/lymbin.go`).
- **Fix:** in `needsPrebuiltLym`, count a run whose tier command fails `gateslot.HoldsTargets` as using the binary. Add a row to `TestInvocationsUseLymBinary` or the gate scenario test.

Tool calls:
1. `git show --stat 1228219e9` and `git diff --stat` over the range: my context lacked which files the last commit and the whole range touch, which I needed to group hunks into changes.
