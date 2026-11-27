```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. **The kit is imported by a `_test.go` file directly in the directory, external or in-package, in any file of several.** `TestTestsUseLymBinary` rows "an external test file importing the kit", "an in-package test file importing the kit among others" and "a second test file importing the kit is enough" (`internal/impactset/tiertests_test.go:97-99`).
2. **Build constraints are ignored.** Row "a test file under another platform's constraint still counts" (`tiertests_test.go:100`).
3. **A non-test file's import does not count, and neither does a directory with no test file.** Rows at `tiertests_test.go:102-103`.
4. **Other imports, the tiergate kit included, do not count.** Row at `tiertests_test.go:101`.
5. **The kit's own directory counts (the `lymbinDir` suffix match).** Row "the kit's own directory" (`tiertests_test.go:104`).
6. **A directory that cannot be read, or a test file that does not parse, counts as using the binary.** Rows at `tiertests_test.go:105-106`.
7. **An import path ending `/internal/testkit/lymbin` from another module counts.** Pinned indirectly by `TestInvocationsUseLymBinary` row `NestedModulePackageResolvedFromItsRunDir`, whose fixture imports `example.com/mod/internal/testkit/lymbin` (`internal/gatecli/prebuild_test.go:137,159`). No impactset row pins it.
8. **`invocationsUseLymBinary` target forms:**
   - no target: `NoTargetRunsWhole`;
   - `...` patterns: `WholePattern` and `SubtreePattern`;
   - an import path: `ImportPathTarget`;
   - relative and absolute directories: `PackageWhoseTestsImportTheKit` and `AbsolutePackageWhoseTestsImportTheKit`;
   - a run directory per invocation: `NestedModulePackageResolvedFromItsRunDir`;
   - the negative case: `PackagesWhoseTestsDoNotImportTheKit`;
   - an empty list: `NoInvocation`.

   All are at `prebuild_test.go:153-161`.
9. **The `.`, `..` and `../x` branches of `isDirectoryTarget`.** No test pins them (`prebuild.go:106-108`). A mutation that drops a branch fails safe, since the target then counts as a non-directory and the tier builds, so no test would catch it.
10. **`needsPrebuiltLym` returns false when the tier asks for no prebuild.** The existing subtest "an untagged tier builds nothing and strips an inherited prebuilt variable" (`test_integration_test.go:1946-1955`) would catch a broken check: `./pkg` is absent and would otherwise count as using the binary.
11. **`runInvocations` skips the build when the targets' tests do not use the binary.** The new subtest "a prebuilding tier whose targeted package's tests do not use the lym binary builds nothing" (`test_integration_test.go:2096-2114`).
12. **`runInvocations` builds when they do.** The existing subtests at `test_integration_test.go:1968` and `2074` pin it. They target `./pkg`, which no fixture in the prime or standalone worktree creates (the only `pkg` written is `guardtiers/pkg`, line 1077). So they build through the unreadable-directory fallback, never through the import rule. The import rule is pinned only at unit level, in item 8.
13. **`runTierCopies` builds once for all copies.** `TestRunTierCopies_BuildsThePrebuiltLymOnce` (`internal/gatecli/copies_test.go:240-276`). Its invocation has no target, so it builds.
14. **`runTierCopies` skips the build when the targets' tests do not use the binary, and `run.tier.PrebuiltLym = false` now runs unconditionally.** No test pins either. Reverting `copies.go:65` to the old `run.tier.PrebuiltLym && ...` condition fails no test.
15. **The "builds none" log line in `needsPrebuiltLym`.** No test pins it. It is read by humans only, so this is no gap.

**Findings**

- **LOW, design.** Location: `internal/gatecli/copies.go:65`. Nothing pins the skip on the `--copies` path. Reverting the condition to always build when the tier asks leaves every test green. Evidence: `copies_test.go:262`, where the only copies test uses `invocations: []invocation{{dir: dir}}` with no target, and the new integration subtest at `test_integration_test.go:2107`, which runs without `--copies`. Fix: add a row or subtest that runs `runTierCopies` with a target directory whose tests do not import the kit, and assert the stand-in `go` recorded no build.
- **LOW, design.** Location: `internal/gatecli/test_integration_test.go:1976,2079`. The CLI-level build subtests pass only because `./pkg` is absent, so they pin the unreadable-directory fallback and not the import rule. No end-to-end row shows a targeted existing package that imports the kit making the tier build. Evidence: no fixture writes `<prime>/pkg` (grep of `test_integration_test.go`), and the fallback sits at `tiertests.go:44-47` in the diff. Fix: write a `pkg/pkg_test.go` that imports the kit into the worktree those subtests target, so they build through the rule they describe.
- **NIT, design.** Location: `internal/gatecli/prebuild.go:106-108`. No row pins the `.`, `..` or `../` branches of `isDirectoryTarget`. A wrongful match would go uncaught only if it also skipped a needed build, so a mutation fails safe and the risk is small. Evidence: the rows at `prebuild_test.go:153-161` use only `./x` and absolute targets.

Tool calls:
1. Grepped `test_integration_test.go` for `pkg` fixtures and the prime setup, to learn whether the existing prebuild subtests target an existing directory. My context showed their targets but not whether any fixture creates that directory.
