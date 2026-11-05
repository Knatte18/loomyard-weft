```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote about the code holds at this worktree's HEAD.

**Findings**

- **MEDIUM**, card 24, the Intent bullet that starts "`baselineFixture.seedScopeRecord` drops its `report` parameter … and its callers in `TestDriftBaseline`, `TestDriftAwait` and `TestDriftVerify` drop the argument."
  - **Summary:** the list of callers leaves out `newVerifyFixture`. At HEAD that function also calls `seedScopeRecord`, and card 24 neither lists it under Edit nor mentions it.
  - **Evidence:** an awk scan of `internal/driftengine/verify_test.go` puts one `seedScopeRecord(` call inside `func newVerifyFixture(t *testing.T, scope ScopeRecord, actions []func(`. The other four are inside `TestDriftVerify`. `verifyFixture` embeds `*baselineFixture` (verify_test.go:98-99).
  - **Why it matters:** card 16 rewrites `newVerifyFixture` "over `commitStore`", but it does not say whether the rewrite still calls `seedScopeRecord`. If the call survives, the driftengine test package stops compiling at card 24.
  - **Fix:** either add `internal/driftengine#newVerifyFixture` to card 24's Edit list and its caller list, or state in card 16 that the rewritten fixture no longer calls `seedScopeRecord`.

**Claims checked that hold**

- `awaitCaptureEnd` takes its sleep as a parameter (wait.go:18), and it waits from one second, doubling to thirty (wait.go:16). So card 9's "one second, then two" matches.
- `statusCommitPathspec` stages the reviews directory and the loom durable directory, as card 15 says (wiring.go:104-109).
- `preflight.runStateExemptRels` exempts `LoomFrictionArchivesRelPattern`, which is `LoomFrictionDirRel()+"-*"`. That matches card 15's archive prefix and the directories `IsLoomFrictionArchive` accepts.
- `verifyRow` exists (verbs.go:21), which supports card 19's "beside `expectRow` and `verifyRow`".
- `newBaselineFixture` sets `Skip: f.configSkip`, and `TestDriftBaseline` holds the row that sets `configSkip`, as card 26 says.
- `TestWire_DarnRecipe` reads `BaselineTable("")` and asserts on `subject_reading` (wiring_darn_test.go:142-146), as card 24 says.
- The `docs/overview.md` sentence card 22 names exists at line 534.
- The `TestLoadConfig` subtest maps that cards 24 and 25 name hold the named fields (config_test.go:444, 450).

**Tool calls**
- Ran an awk scan of await_test.go, verify_test.go and baseline_test.go for the functions that enclose each `seedScopeRecord` call and each `configSkip` use. My context had only per-file counts, not the enclosing functions.
