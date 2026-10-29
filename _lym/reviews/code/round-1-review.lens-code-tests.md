```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds, the test that pins it, or none.

**Per-behaviour list**

The diff is test-only: it adds one subtest, `fails with the file and line of a misplaced marker`, in `TestVerifyGate_RoundScenario` (`internal/loomshed/verifygate_integration_test.go:379-392`). It changes no production code, so the subtest is the only behaviour to check. The behaviours it covers are below.

- **Round form refuses a misplaced `//lym:guard` marker, naming `a/a_test.go:5` and "directly above".**
  - The subtest's `f.runGate(t, nil)` assertion pins it, at lines 381-384.
  - It would fail if the guard scan were skipped, or if the refusal lost its location or its way forward. The refusal text from `guardScanRefusal` (`internal/gatecli/cli.go`, GuardMisplaced case) contains "directly above".
  - Pinned.
- **Darn form with the first recorded pass as merge base and a not-scoped tree scope refuses the same marker.**
  - The `f.diffGate(toldCommand, baseOf(firstPass.Commit), notScoped)` assertion pins it, at lines 386-389.
  - It would fail if the narrowed-diff guard scan stopped running on the darn form.
  - Pinned.
- **No skip: the subtest must run.**
  - It has no `t.Skip` call, and the `if !t.Run(...) { return }` wrapper matches its neighbours.
  - Pinned.
- **The next subtest's rewrite of `a/a_test.go` clears the marker for later steps.**
  - The next subtest, `fails on a wrapped new comment before any test runs`, commits `a/a_test.go` as a clean file, and it asserts `a/a.go:3`. A marker left behind would fail it.
  - It is covered indirectly by that subtest and the ones after it.
  - Pinned.

**Findings:** none.

**Test economy.** The subtest duplicates no existing coverage in `internal/loomshed/verifygate_integration_test.go`. The neighbouring subtests cover the wrapped comment and the overview, not the guard marker. It does not assert on stencil prose.

**Tool calls:** none. The manifest already held the diff, the test file and the `internal/gatecli/cli.go` refusal text.
