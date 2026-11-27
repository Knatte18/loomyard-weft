```yaml
lens: code-tests
outcome: reported
```

**Per behaviour — the test that pins it**

1. A marked key runs a marked test whose tier the key does not list — `TestGateTest_Scopes`, `internal/gatecli/scopes_integration_test.go:498` (`wantArgs` expecting two invocations and exit 0); a reinstated refusal would fail it. Pinned only partly: see finding 1, it does not pin that the tier1 test runs under tier1's own command/settings.
2. A marked test of a tier the hub does not declare is still refused — the planTierRuns row "a guard group whose tier is undeclared" (`internal/gatecli/tiers_test.go:590-595`) drives the same branch (`tiers.go:229-231`) via `scopeAll` + `request.guards`; the marked path reaches that branch via `sc.guardStep`, and the branch is scope-form independent. Breaking the branch fails that row, so the behaviour is covered; no marked-scope row exists, but none is needed for coverage.
3. Removed refusal / removed `markedNames` helper — the rewritten case at `scopes_integration_test.go:498` pins the absence of the refusal; the old `wantRefusal` assertion was replaced because the behaviour was intentionally withdrawn, not loosened.
4. doc.go / template.yaml text — static prose, exempt.

**Findings**

1. LOW, design — `internal/gatecli/scopes_integration_test.go:493-498`: the comment claims the tier1 marked test runs "under tier1's command", but nothing asserts it. `wantArgs` compares only each stand-in record's `Args` (`test_integration_test.go:495-503`), and tier0 and tier1 share the identical stand-in command `{targets} {args}` (`test_integration_test.go:640-641`), so the test would still pass if the guard step ran the tier1 group with tier0's command, env (`GOFLAGS`) or slots. Fix: add an assertion on the guard run record via the existing helper — `guardRunRecordOf(t, out, 1)` with `GuardStep` true, `Tier` 1 and tier1's `Slots` ("all" → the pool's size), which the fixture distinguishes from tier0 (3-slot vs all-slot).

Tool calls:
- Read `runningGate.tierArgs` in `test_integration_test.go` — to confirm whether `wantArgs` checks command or only args; context lacked its body.
