```yaml
lens: code-correctness
outcome: reported
```

**Per-branch list (code-correctness): all cases ran correct, no findings.**

- **Commit step.**
  - The committed file is `package a` (line 1), blank (2), `import "testing"` (3), blank (4), `//lym:guard` (5), `var misplaced = 1` (6).
  - The marker therefore sits on line 5 above a non-test declaration. `guards.go:185` reports `GuardMisplaced` with `Line: markerLine`, and `File` is the worktree-relative, slash-separated path (`guards.go:51`), so `a/a_test.go:5` is the expected location.
  - `cli.go:219` prints that location with "directly above", so both asserted substrings match the refusal. Correct.
- **Round form: `f.runGate(t, nil)`.**
  - With a nil merge base the gate takes the whole-module path; the open-risk note in the decision record says the guard scan still runs over the module.
  - Case run: a misplaced marker, `passed=false`, findings holding `a/a_test.go:5` and "directly above". The condition `passed || !Contains(...) || !Contains(...)` errors on any miss. Correct.
  - `runGate` calls `t.Fatalf` on a gate error. A gate error would therefore stop the subtest, not be swallowed. Correct.
- **Darn form: `f.diffGate(toldCommand, baseOf(firstPass.Commit), notScoped)`.**
  - `firstPass` and `baseOf` are declared before this point in the scenario. `firstPass` is already used at lines 304 and 359 in the same shape. `toldCommand` and `notScoped` are declared at lines 282 and 286, before this subtest at line 379. The signatures match `diffGate(command, mergeBase, treeScope)`.
  - Case run: `err == nil`, `Passed == false`, findings naming the marker. The `||` chain also fails on a non-nil `err`. Correct.
  - Case not run: the `got.Passed` condition cannot tell a guard-scan refusal from some other failure of the told command, but the substring checks pin the specific message. Correct.
- **Early `return` on failure.**
  - It matches the neighbouring `if !t.Run(...) { return }` blocks at lines 394 and 419.
  - Case run: the subtest fails, so the later steps are skipped. They would otherwise run on a tree holding a misplaced marker. Correct.
- **State left for later steps.**
  - The next subtest at line 394 rewrites `a/a_test.go` to the clean `TestA` file in its first commit, so no misplaced marker remains. Correct.
  - Case run: the successful path leaves HEAD with the marker commit, and the next subtest's `runGate(t, nil)` expects a failure naming only `a/a.go:3`. The clean rewrite is committed before that call, so the guard scan passes. Correct.
- **Verify-log and record state.**
  - The new subtest asserts no log count, and a guard-scan refusal runs before any command. The next subtest's `logsBefore` is read after its own commit, so the earlier refusal does not skew it. Correct.
- **Scope and tags.** The diff is one test-only file with no skip, under the file's existing `tier1` tag, and it covers both the round form and the darn form.

**Tool calls**
1. `grep` in `internal/impactset` for `GuardScanError{` and `File`: my context lacked the guard scan's file-path form (relative and slash-separated) and the exact marker line it reports.
