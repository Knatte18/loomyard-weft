```yaml
lens: code-affected-set
outcome: reported
```

My lens was code-affected-set: what the new test file forces outside its own lines, and whether it fits the repository's rules, the rubric's "Commit messages and departures" and "Declared PATTERN changes landed".

**Affected set, per change.** The diff adds one test file, `internal/loomcli/amendduringwebster_test.go`, and changes no production symbol, signature, output, key, message or file shape. Nothing calls, reads or asserts a test function, so there are no callers, docs, stencils or fixtures for it to leave wrong.

- **Shared helpers it calls:** `newDecisionFake`, `goodDecisionInput`, `planApprovedReader`, `writeStatusFixture` and `productJSON`. It uses each through its existing signature and changes none of them. Covered.
- **Tree-sitter link guard:** the test imports `internal/planglyph` into the `loomcli` test binary. `loomcli` is already on the allow list (`cmd/lym/treesitterlink_integration_test.go:21`), and `wiring.go` and `validate.go` already import `planglyph`. Covered.
- **PATTERN-test-speed, tier0 spawns nothing:** the file declares `tiergate.Require(t, 0)` and calls no `gitexec`, `exec`, `hubforge`, `gitkit`, `lymbin` or `tmuxkit` export. `plankit.Repo` only writes files (`internal/testkit/plankit/plankit.go:105-110`). `NewPlanGate` with `planglyph.NewIndex` runs quarry in-process, and `loomshed`'s tier0 gate tests already run it the same way (`internal/loomshed/gates_test.go:588`). Covered.
- **PATTERN-test-speed, parallel, no sleeps, no fixed clock:** `t.Parallel()` is called, there is no sleep, and no wait on real time. Covered.
- **PATTERN-test-isolation:** the test writes only under its own `t.TempDir` directories (the anchor, the repo and the run directory). Covered.
- **PATTERN-test-only-code:** there is no new production function. Covered.
- **File header and doc comment conventions:** the header is present, a blank line separates it from `package`, and the doc comment opens with the test's own name. Covered. Line breaks are the gate lint's job and are not judged here.
- **Commit messages and departures:** the three commits carry no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. Nothing needs one, since no existing check is loosened and no departure is taken. Commit 7da9d99's subject starts with `fix:`, but the "fix" is splitting a comment, not a claim about a bug. Covered.
- **Declared PATTERN changes landed:** the directive declares none, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing is due.
- **Stencil wording:** no stencil is changed. Not applicable.

**Findings:** none.

Outside my lens, for the other forks or the handler: the decision fake's `implementerSeam` uses its own separate temp anchor (`internal/loomcli/decision_test.go:253`), not the test's `anchorPath`. So `decisionVerb`'s implementer reads never see the planted Webster state. Also, the test's `shedengine.Goto` call passes no `Admitted` hook, so loom's real goto hook is not driven.

Tool calls:
1. Grepped which `loomcli` files import `planglyph`, the tree-sitter allow-list entries, and `planglyph`'s spawn and quarry use. My context lacked the link-guard list and whether the index spawns a process at tier0.
