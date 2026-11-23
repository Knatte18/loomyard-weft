```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the `diff.go` untracked-file branch and the three added or changed test assertions.

no findings

- **`diff.go` untracked branch.** `RunGit` returns a non-zero exit as the exit code and not as an error, and `git cat-file -e <base>:<path>` on a missing path exits 128, so `err` stays nil and the file keeps no base side. A path the base holds exits 0 and gets `basePath = path`, which `readChange` reads with `git show`. With `Lint(dir, "")`, base is `"HEAD"`. An ordinary untracked file therefore still has an empty base, and `untracked.go:3` is still reported.
- **Untracked-file row in `TestLint_OverARepository`.** After `git rm --cached` and a commit, `git diff --name-status <withIndexless>` lists `D indexless.go`, which `parseNameStatus` drops. The untracked pass then adds it with a base side, so no line counts as new and none is reported. Only `renamed.go:7` and `untracked.go:3` remain, as the row expects. With the defect in place, `indexless.go:3` would be reported and the row would fail, so the row catches the regression.
- **Verify gate (F2).** The untracked wrapped file is written before `runGate` and removed right after it. With the defect in place the gate would use `Lint` and report `a/untracked.go:3`, so the `strings.Contains(findings, "untracked.go")` check would fail. Under `LintCommitRange` the file is not seen. The one gap is that a `t.Fatal` inside `runGate` would leave the file behind, but that ends the test anyway.
- **F3 and F4.** These change names and comments only, with no behaviour change.
- **Callers.** `changedGoFiles` and `readChange` keep their signatures. `LintCommitRange` takes the `workingTree == false` path, which the fix left alone.

Further tool calls: none.
