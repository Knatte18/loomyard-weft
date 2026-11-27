```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the prebuild-skip diff. I ran each changed function and branch on a concrete input.

**Per function or branch: case run, verdict**

- **`needsPrebuiltLym`, tier without `prebuilt_lym`:** returns false and logs nothing. Correct.
- **`needsPrebuiltLym`, tier with `prebuilt_lym` and targets `./quiet` (tests import only `testing`):** `invocationsUseLymBinary` returns false, so it logs once and returns false. Correct.
- **`invocationsUseLymBinary`, nil invocations:** returns false. Correct, because both callers return early when there are no invocations.
- **`invocationsUseLymBinary`, invocation with empty targets** (a whole run whose `all_targets` is empty): returns true. Correct.
- **`invocationsUseLymBinary`, targets `./internal/foo/...` and `./...`:** returns true. Correct, if conservative.
- **`invocationsUseLymBinary`, import-path target `example.com/x` or bare `pkg`:** `isDirectoryTarget` is false, so it returns true. Correct.
- **`invocationsUseLymBinary`, target `./internal/loomcli/x_test.go` (a file):** `ReadDir` fails, so it returns true. Correct.
- **`invocationsUseLymBinary`, target `./user/` with a trailing slash:** `filepath.Join` cleans it to `user`. Correct.
- **`invocationsUseLymBinary`, `-C sub` with target `./p`:** the target resolves against root/sub, the same directory `runInvocations` uses. Correct.
- **`invocationsUseLymBinary`, guard step:** `guardStepRun` fills packages `./dir` at execution time (`run.go:617`), before `runInvocations` checks them. Correct.
- **`invocationsUseLymBinary`, explicit targets `./plain` on a tier whose command holds no `{targets}` or `{test_targets}`:** returns false, but the command runs whole. **Wrong**, see F1.
- **`invocationsUseLymBinary`, explicit targets `./plain` on a command like `go test {targets} ./cmd/...`:** the literal packages in the command are never read, so it returns false. **Wrong**, see F1.
- **`isDirectoryTarget`:**
  - `.`, `..`, `./x`, `../x` and `/abs` are directories; `x`, `-run`, `.\x` on Windows and `.x` are not.
  - A Windows `.\x` therefore builds, which is the safe side. Correct.
- **`TestsUseLymBinary`:**
  - The kit's own absolute directory returns true.
  - `/a/foointernal/testkit/lymbin` returns false because of the `/` prefix.
  - A relative `.` gets no suffix match and falls through to `ReadDir`.
  - Missing directory returns true.
  - Unparsable import block returns true.
  - A body error after the imports parses under `ImportsOnly`, so the file's imports decide.
  - A nested module's `.../internal/testkit/lymbin` import returns true, an over-build that is harmless.
  - A non-test `.go` file importing the kit returns false.
  - Correct.
- **`runTierCopies`, `prebuilt_lym` set, check true:** builds once and exports the binary; the copies then run with `PrebuiltLym=false`. Correct.
- **`runTierCopies`, `prebuilt_lym` set, check false:** env stays as it is, and `gateEnv` (`run.go:912`) has already stripped any inherited `LYM_PREBUILT_LYM`. `PrebuiltLym=false` is set, so the copies neither re-check nor re-log. Correct.
- **`runTierCopies`, build fails or ctx is cancelled:** the paths are unchanged. Correct.
- **`runTierCopies`, `prebuilt_lym` set with no invocations:** skips the build and sets the flag false. This is harmless, because `runInvocations` returns early on no invocations. Correct.
- **`runInvocations`, check false:** no build, no export; the env was already stripped, and the timeout, retry and cleanup paths are untouched. Correct.
- **`runInvocations` on a retry:** the check runs again and logs again. That is redundant, not wrong.

**Findings**

F1. MEDIUM, design. Location: `internal/gatecli/prebuild.go:86-102` (`invocationsUseLymBinary`).
Summary: the check reads `invocation.targets` even when the tier's command does not run those targets, so it can skip the build on a call that runs the whole set. This contradicts the board entry's "A call over the whole set still builds it".
Evidence:
- `internal/gatecli/tiers.go:293-298`: under explicit targets, `scopeInvocations` returns `targets: request.targets` with `ignoresTargets = !gateslot.HoldsTargets(tier.Command)`.
- The doc at `tiers.go:285-286` says "A command holding neither {targets} nor {test_targets} runs once, whole. It does so under explicit targets."
- The scenario fixture's `tier2: standIn+" whole"` (`test_integration_test.go:642`) shows this command shape is supported.
- Effect: `lym gate test --tier N ./plain` on a prebuilding tier whose command is, say, `go test -tags tierN ./...` exports no binary. Each `lymbin`-importing test binary then compiles `lym` itself (`lymbin.go` `buildOnce`/`compile`), which is a per-package build where there used to be one.
- The same applies to a command that names literal packages beside `{targets}`.

Fix: when `!gateslot.HoldsTargets(run.tier.Command)`, count the invocation as using the binary, as is already done for an invocation with no target. Add a row for it to `TestInvocationsUseLymBinary`, which would mean passing the tier or its command in. A command that holds a placeholder and also names literal packages cannot be read without parsing the command, so build whenever the command holds package words besides the placeholders, or document that limit.

Tool calls:
1. `grep` for `guardStepRun(` and `guardStep:` in `internal/gatecli`: to confirm the guard step's targets are filled before `runInvocations` checks them, which the manifest did not show.
