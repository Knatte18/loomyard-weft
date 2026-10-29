```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: for each thing the diff changes, the set of readers, assertions and docs I found, and whether the diff leaves it right.

**Per change:**

1. **A new subtest in `TestVerifyGate_RoundScenario`, placed after the read-error table and before the wrapped-comment subtest (`internal/loomshed/verifygate_integration_test.go:379-393`).**
   - **Fixture helpers.** The subtest calls `f.commitFiles`, `f.runGate`, `f.diffGate`, `toldCommand`, `notScoped`, `baseOf` and `firstPass`. All are defined earlier in the same function or file, with matching signatures (lines 140-170, 230, 257, 282, 286). Covered.
   - **State the next subtest relies on.** The wrapped-comment subtest at line 394 commits a clean `a/a_test.go`, which overwrites the misplaced marker. Later subtests, including the overview subtests that run `runGate(baseOf(parent))`, therefore see no misplaced marker. Covered.
   - **Net diff for the darn form.** Any later darn call from `firstPass.Commit` sees the misplaced commit and its clean rewrite, so the net `a/a_test.go` diff is clean. Covered.
   - **Verify-log count.** Neither the new subtest nor the wrapped-comment subtest depends on the log count left by the other. The new subtest records no pass, and a failed gate leaves no green record. Covered.
   - **Early `return` on failure.** It matches the neighbouring `if !t.Run(...) { return }` pattern (lines 232, 246, 394). Covered.
   - **Asserted text.** `directly above` and the `a/a_test.go:5` location match `guardScanRefusal` (`internal/gatecli/cli.go:219`). Line 5 is `//lym:guard`, which sits above `var misplaced = 1`, a non-test declaration. Covered.
   - **The test's own doc comment.** `TestVerifyGate_RoundScenario` says "through the told command, the lint and the refusals". The new step fits that wording, so no edit is forced. Covered.

2. **Docs.**
   - The decision record's D2 says `internal/loomshed/verifygate.go` and `internal/gatecli/doc.go` already document this behaviour. The diff changes no observable behaviour or invariant, so `CLAUDE.md`'s docs-in-the-same-commit rule does not apply. Covered.

3. **`PATTERN.md` and `pattern/`.**
   - `PATTERN-test-speed`: the file keeps its `tier1` tag, nothing is skipped, and the subtest runs in the shared sequential scenario that already documents why it is not parallel. Covered.
   - `PATTERN-test-isolation`: the subtest uses the scenario's own scratch repo and `gitkit`. Covered.
   - No PATTERN entry declares a change, and the diff touches neither `PATTERN.md` nor `pattern/`. Covered.

4. **Stencils.** The diff touches none. Not applicable.

5. **Commit message and departures.**
   - The message carries no `Pattern-Departure:` trailer, and none is needed because the diff departs from no entry. Covered.
   - The diff is test-only, with one file changed, which matches the board entry's scope.

**Findings:** none.

**Tool calls:** none. The manifest already held every definition and call site this lens needed.
