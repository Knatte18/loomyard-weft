# Fixer-report — round 1, code

## Changes

- **F1 (BLOCKING, fixed as proposed):** `shortnamebinding_clone_integration_test.go` carries a comment above `ResetWithDisagreeingShortnameKeepsHub` saying that subtest and `ResetKeepsRecord` skip `t.Parallel` because they share one existing hub (`firstParent`, `first`) and run in order.
  The first draft sat above all the subtests and was too broad, per the self-check; it was narrowed in the follow-up commit.
- **F3 (MEDIUM, fixed as proposed):** `warpbinding_clone_integration_test.go` gains `TestCloneHub_ResetRefusalKeepsExistingHub` with rows for a warp-binding mismatch (asserts `ErrWarpBindingMismatch`) and the old-order guard (asserts `refusing to bootstrap`).
  Each row plants a sentinel file in the existing hub and asserts it survives the refused `--reset`.
  The mismatch row uses two warps that share a basename so both URLs derive the same hub path.
- **F4 (LOW, fixed as proposed):** the comments in `livestate_refusal_selftest_test.go`, `livestate_verbs_test.go` and `livestate_matrix_test.go` name `checkResetTarget` or the offline pre-flight, with no line numbers.
- **F5 (NIT, fixed as proposed):** the shared-block comment in `clone.go` says the offline check of the two-argument form repeats there, and the asymmetry sentence sits with the one-argument remark.
- **F2 (MEDIUM, decision):** left out of my own judgment, as the instructions require, until mail #787 from `ly:LOOM` delivered the parent's recorded decision: no narrowing, and `--subpath` is checked before `--reset` removes anything.
  I acted on it as the parent's instruction:
  - `resetsubpathcheck.go` (new) adds `checkSubpathBeforeReset`, which stages shallow clones of both remotes under `cwd` and applies the adopt-path rule (recorded anchor on the weft) and the create-path rule (directory present in the warp) before the hub is touched.
  - `clone.go` calls it only when `Reset` is set, `--subpath` is explicit and the hub exists.
  - `recordsprobe.go` shares `fetchWeftBranchTip` with the check, and `probeMainWeftHasPath` now uses it.
  - Tests: `SubpathDisagreesWithRecordedAnchor` and `SubpathAbsentFromWarp` rows in the refusal test, plus `TestCloneHub_ResetWithSubpathPresentInWarp` guarding against over-refusal.
  - Docs: the `CloneHub` doc comment and `doc.go` state the subpath check.
- **Gate repair:** the gate refused seven comments I had wrapped at fixed columns, in `clone.go`, `resetsubpathcheck.go` and four test files.
  Each now holds one sentence per line, with no change to its wording.
- **Verify repair:** `TestNoDestructiveBypass_FabricengineProductionSource` flagged `RemoveAll(` in the new `resetsubpathcheck.go`.
  `cmd/lym/destructiveguard_test.go` now allowlists that file with a reason, as it already does for the probe's identical removal of its own throwaway directory.
- **Second verify finding, no change made:** `TestGateTest_DotnetFixture` in `internal/gatecli` failed when the .NET build of its fixture aborted with `Internal CLR error (0x80131506)`, exit 134.
  The round changes only Go files in `internal/fabricengine` and `cmd/lym`, and the test drives a C# fixture, so the round does not reach it.
  Re-running `lym gate test ./internal/gatecli --tier 1` passed (180 passed, 0 failed), so I took it as a transient crash of the .NET runtime on this machine.
- **Self-check follow-ups:** `warpProbeDirPrefix` comment, the `recordsprobe.go` header and the `warpbinding_clone_integration_test.go` header now cover the new users and tests.

Commits: `134a2774` (F1), `0ff2b753` (F4), `b29e887d` (F3, F5 and #787), and one follow-up commit for the self-check fixes.
Tier 1 of `./internal/fabricengine` passes with the new tests running (`lym gate test ./internal/fabricengine --tier 1`: 1760 passed, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `warpProbeDirPrefix` comment names only the probe: fixed as proposed.
- fix-consistency, LOW, `recordsprobe.go` header omits the shared fetch helper: fixed as proposed.
- fix-consistency, LOW, `warpbinding_clone_integration_test.go` header omits the new refusal tests: fixed as proposed.
- fix-consistency, LOW, the pre-check does not mirror the stale `.fabric-anchor` hard error: deferred, see below.
- fix-proposal, LOW, the F1 comment names subtests that do not share the hub: fixed with my own change (the comment moved above the two subtests that share it).
- fix-affected-set, LOW, `doc.go:512` still states the old removal order: fixed as proposed.
- fix-proposal, no findings for F3, F4 and F5.

## Disputed

None.

## Deferred

- **Stale `.fabric-anchor` marker under `--reset` (self-check, fix-consistency, LOW):** a `--reset` against an old-layout weft with a stale `.fabric-anchor` and no `.lym-anchor` still reaches the stale-marker hard error after the hub is removed.
  It is raised whatever `--subpath` says and is not an argument check, so covering it means widening the pre-check beyond the recorded decision, which names `--subpath` only.
  Parent to decide: record a decision extending the pre-check to the stale marker, or accept it.
