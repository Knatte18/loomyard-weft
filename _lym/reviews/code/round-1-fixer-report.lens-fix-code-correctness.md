```yaml
lens: fix-code-correctness
outcome: reported
```

Findings from the code-correctness lens:

**MEDIUM**: `run` in `internal/gatecli/redfirst.go`, the per-package loop.
- Summary: the per-package split (F13) has no test that drives `run`, so nothing fails if it reverts to one union run.
- Evidence: the added tests cover `testsByPackage` (a pure helper) and `judgeRedFirst`. `TestGateRedFirst` seeds only the `calc` package. With the loop reverted to a single `-run` union, every added test still passes.
- Why it matters: the finding's premise (a same-named test in another package failing on the base turns a green run into an unjudged halt) is untested.

**LOW**: `pattern/PATTERN-gate-slot.md`, the edited "`Verify` takes no gate slot" bullet.
- Summary: "`lym gate test --wait` calls, or the `lym gate red-first` call, which take their own slots" asserts that `lym gate red-first` takes its own slots.
- Evidence: the added PATTERN bullet and the `verify.go` doc say the call takes them "through the in-process tier run". I could not confirm from context that `runTest` acquires gate slots. If it does not, the claim is wrong.

**LOW**: `baseOverlay` and `holdsGoFileAtOrAbove`.
- Summary: the F4 refusal is broader than a missing embed or read. Any changed non-Go file at or below a directory that holds a `.go` file is refused, for example a `README.md` beside `doc.go`, or `contracts/**/*.md` when a Go file sits in `contracts/`.
- Evidence: the rule is `held` from `goFiles(dir)` at each ancestor, with no check that the file is embedded or read. A doc-only change in such a directory exits `RedFirstUnjudgedExit` and sends the writer to the parent.
- Note: this matches the reviewer's F4 intent (refuse a known false green), so it is a stated trade-off, not a defect. The `NonGoFileBelowAPackageDirectoryIsUnrestorable` row pins the behaviour.

What I checked and found correct:
- `NamedFailedTests`: every row of the new table gives the wanted result, including the `-1` tier for `vet tier1` and the cut-line break. The two added `judgeRedFirst` rows fail if `FailedTests` is restored.
- `baseOverlay` rows: the expected `unrestorable` values hold. `sub/notes/readme.md` is left alone because `sub` holds no Go file in the `memoryTree`.
- `changedTestsAt`: the `sub/c` row fails under the negated condition, since only `sub/c/c_test.go` selects the package.
- `redFirstFailure`: the exit-5 case is non-terminal and sits after the timeout and crash cases, so a timed-out run is still terminal.
- Callers: `baseOverlay` is called only in `run` and in the test. No remaining reference to `loomshed.BugLabel` or `loomshed.NoTestSeamTitle` exists outside the removed declarations.

Further tool calls (1 of 6):
- One `grep` over `cli.go`, `shedrecipe/doc.go`, `darnrecipe_test.go` and `darn-template-write.md` for stale red-first text, and over the repo for `baseOverlay(` callers and the removed `loomshed` constants. My context held none of these files, and the grep found no stale scoper or exit-0 wording.
