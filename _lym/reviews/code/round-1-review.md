---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/commentlint/diff.go#changedGoFiles
    summary: With a base other than HEAD, a path the base holds that is untracked in the working tree is linted with no base side, so its legacy comment lines are all reported as new
    fix: "In changedGoFiles, when workingTree is true, give each untracked path a basePath equal to its path when baseRevision holds it, checked with `git cat-file -e <baseRevision>:<path>` through gitexec. Add a row to TestLint_OverARepository that runs `git rm --cached` on a file holding a legacy wrapped comment, commits, and asserts Lint against the earlier base reports no finding for that file."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/verifygate_integration_test.go
    summary: No test pins that the verify gate lints the committed range and ignores the working tree, so reverting verifygate.go:146 to commentlint.Lint passes every loomshed test
    fix: "In an existing comment-lint case of the verify gate's integration test, leave an uncommitted or untracked `.go` file with a wrapped comment in the worktree before calling the gate, and assert the gate's findings do not name that file."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/lintcomments_integration_test.go:104
    summary: "The subtest name \"--base lints the committed range\" and the test doc's \"a commit range\" still describe the old meaning of lint-comments --base, which now lints the working tree against that commit"
    fix: "Rename the subtest at line 104 to say --base lints the working tree against a base commit (on a clean tree it sees the committed change), and change \"a commit range\" in the TestLintCommentsCmd_OverARepository doc comment at line 21 to name a base commit."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/commentlint/lint_integration_test.go:1
    summary: The file header says the file drives Lint over a real repository, but it now drives LintCommitRange too
    fix: "Change the header to read \"lint_integration_test.go drives Lint and LintCommitRange over a real repository.\""
---

The diff takes the board entry's first option: `commentlint.Lint` now always reads the working tree, against the told base or HEAD.
The new `commentlint.LintCommitRange` keeps the base-to-HEAD comparison for the verify gate.
An empty base gives exactly the git arguments and reads of the old empty-base path, so the gate's `comments` step (`internal/gatecli/failurereport.go:118`) and `lint-comments` without `--base` are unchanged.
`LintCommitRange` reproduces the old non-empty-base path exactly, and `internal/loomshed/verifygate.go:146` is the only caller that needed it.
The `Root-Cause:` trailer names the defect the diff removes.
The summary's claims hold, and its two-way door with a `lint` blast radius fits: no record format, config meaning or outward push changes.
No assertion that existed at the review base is loosened.

### [LOW] An untracked file the base holds is linted as wholly new

**Location:** internal/commentlint/diff.go#changedGoFiles

**Issue:** `git diff <base>` lists a path that the base holds and the index lacks as `D`, and `parseNameStatus` drops it.
`ls-files --others` then adds the same path with no base side, so `readChange` reads an empty base text and every comment line counts as new.
I reproduced the listing in a scratch repository: `git rm --cached a.go`, commit, then `git diff --name-status <base>` printed `D a.go` while `ls-files --others` printed `a.go`.
The old `--base` form never listed untracked files, so `--base` gains this false positive.
The empty-base path has always had it, for a file removed from the index with `git rm --cached`.

### [LOW] The verify gate's switch to LintCommitRange is untested

**Location:** internal/loomshed/verifygate_integration_test.go

**Issue:** Every comment-lint case of the verify gate commits its wrapped file before calling the gate, so no case tells the committed range from the working tree.
Reverting `verifygate.go:146` to `commentlint.Lint` would pass every loomshed test.

### [LOW] A loomcli subtest name still describes the old --base meaning

**Location:** internal/loomcli/lintcomments_integration_test.go:104

**Issue:** The name `--base lints the committed range` and the doc's "a commit range" describe `--base` as the commit range.
The verb now lints the working tree against that commit, and the subtest at line 120 shows the two readings differ.

### [NIT] The commentlint test file header names only Lint

**Location:** internal/commentlint/lint_integration_test.go:1

**Issue:** The header says the file drives `Lint`, but the subtests at lines 57 and 73 drive `LintCommitRange` too.

## Rejected

- The mutation survivor at `internal/loomcli/lintcomments.go:70` is the `--base` flag's help string, static prose no test can reasonably assert, so it is no finding.
- The code-scope lens found every change traced to the board entry's goal, or forced by a traced change, and raised nothing to reject.
