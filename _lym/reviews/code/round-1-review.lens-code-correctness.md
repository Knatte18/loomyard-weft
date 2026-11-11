```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff. Each entry below is a function or branch, the case I ran, and whether it came out correct.

**List**
- `planTierRuns` targets refusal (`tiers.go:142`):
  - Case: `testRequest{targets:[./a ./b]}` with tier0 `make test {args}`, so `wholeTiers=[0]` and `kept=[0]`. Correct: it refuses with the exact message the test pins, before any slot.
  - Case: `--tier 1 ./a` where tier1 holds `{targets}`. Correct: `wholeTiers` is empty and 0≠1, so no refusal.
  - Case: tier0 whole beside tier1 taking targets. Correct: no refusal, and the notice names tier0.
  - Case: no targets, so `sc.form` is not `scopeTargets`. Correct: no refusal, and a placeholder-free tier still passes with a notice.
  - Case: `--gate X ./a`. Correct: `withGateScope` returns early when targets are present, so the form is `scopeTargets` and the refusal applies.
  - Case: an `{args}` refusal on a tier in the same call. Correct: it fires first, inside the loop.
- Notice moved from `runListed` to `planTierRuns` (`tiers.go:146`):
  - Correct for all the run's tiers.
  - Side effect (LOW, consistency): the notice now prints at plan time. It also prints for a run later blocked by the build, as the `test_integration_test.go` change shows. It also prints before a later guard-step refusal in `planTierRuns` (`tiers.go:166-190`), so stderr can hold a "runs whole" line followed by an error.
- `scopeInvocations` (unchanged logic, return value now only feeds `wholeTiers`):
  - Case: a card scope with narrower modules and a placeholder-free command returns `runsWhole, true`. This warns and does not refuse, matching the board entry's "refuse or warn".
- `unrunProjectLines`:
  - Case: the issue's setup, tier0 `dotnet test Fast/Fast.csproj {filter}` and a card selecting `Slow/Slow.csproj`. It collects `Slow/Slow.csproj`, `namesProject` sees the `.csproj` word `Fast.csproj` and returns false, so it warns. Correct.
  - Case: `lowestTierReaching` with no tier1 declared. Correct: it falls back to the config-step wording.
  - Case: a `Whole` or `NoTestClass` share. Correct: `modules` is empty, so there are no lines and no false warning.
  - Case: non-dotnet scoper. Correct: the `cfg.Scoper == "dotnet"` guard skips it.
  - Case: the same project in several tiers. Correct: sort plus `Compact` gives one line.
  - Case: a call that ends in a halt returns before the lines print. Acceptable.
- `runReachesProject` and `namesProject`:
  - Case: empty `invocations` returns false. Correct.
  - Case: empty words. `!namesAny` is true, so everything is reached. This is the documented assumption.
  - Case: `"A\A.csproj"` with quotes and backslashes. Correct: it is trimmed and normalised.
  - Case: a `.sln`. Correct: it returns true.
- `lowestTierReaching`:
  - Case: `Atoi` error on a malformed key gives number 0 and a duplicate tier0. Keys are validated elsewhere, so unreachable in practice.
  - Case: manual tiers are skipped. Correct.

**Findings**
1. MEDIUM, design, `dotnetscope.go:184-204` (`namesProject`): a command that runs a built assembly, such as `dotnet test Fast/bin/Debug/Fast.dll`, names no `.csproj`, `.fsproj`, `.vbproj` or solution file. `namesAny` stays false, so the function returns true and says the command reaches every project. The board entry says tier0 "runs only the fast test assembly", so this is plausibly the reported setup. The warning would then stay silent and the original defect would remain. The same applies to `make`-style wrappers. Fix: treat a command that names a `.dll`/`.exe` or any other non-project target as not reaching an unnamed project. Alternatively, count "reaches all" only for a bare `dotnet test` with no path words.
2. LOW, design, `dotnetscope.go` (`namesProject`): the match is by file name only, via `path.Base`. Two projects with the same file name in different directories, for example `A/Tests.csproj` and `B/Tests.csproj`, count as reached when only one is named. This gives a false negative. Fix: compare the normalised relative path, falling back to the base name only when the command word has no directory.
3. LOW, consistency, `tiers.go:146-148`: the plan-time "runs whole" notice is also emitted for a plan that later fails in the guard/vet branches. The old run-time notice did not do this. Fix: print the notices after the last refusal in `planTierRuns`, or just before `return runs`.

**Tool calls:** none made. The manifest context was enough.
