```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to a goal point of the board entry and each goal point to the changes that build it.

**Decisions.** The answer key is the decision record alone. Its `## Goal` holds one board entry with one goal point, and the record has no `Added after Discussion` entries.
- **G1:** `lym loom lint-comments --base` gives feedback on uncommitted changes. The entry allows three ways: lint the working tree, add a flag that does, or document the HEAD-only reading in the verb's help.

**Changes and what they trace to:**
- **C1:** `commentlint.Lint` diffs the working tree against the base, with HEAD standing in for an empty base, and adds untracked `.go` files. This is the `Lint` and `lintChanges` hunk in `internal/commentlint/lint.go` and the `workingTree` parameter in `changedGoFiles` and `readChange` in `internal/commentlint/diff.go`. It traces to G1, the first option.
- **C2:** an untracked file that the base revision holds gets its own path as its base side, through the `cat-file -e` probe in `internal/commentlint/diff.go` `changedGoFiles`. C1 forces it: once the working-tree form can take a base other than HEAD, an untracked path can exist at that base.
- **C3:** the new `commentlint.LintCommitRange`, and `loomshed.NewVerifyGate` calling it (`internal/loomshed/verifygate.go:146`). C1 forces it: `Lint` changed meaning, and the verify gate keeps the committed base-to-HEAD reading it had before.
- **C4:** docs and help. These are the `internal/commentlint/doc.go` package doc, the doc comments in `diff.go` and `lint.go`, the `lint-comments` Long help and `--base` flag text in `internal/loomcli/lintcomments.go`, the `lint-comments` paragraph in `docs/overview.md`, and the `pattern/PATTERN-gate-slot.md` entry line. C1 and C3 force them, since each text described the old HEAD-only reading or the old single caller.
- **C5:** tests. These are the new working-tree, commit-range and untracked-at-base steps in `internal/commentlint/lint_integration_test.go`, the renamed subtest and the new uncommitted-edit step in `internal/loomcli/lintcomments_integration_test.go`, and the untracked-file assertion in `internal/loomshed/verifygate_integration_test.go`. C1 to C3 force them.

**Decision to changes:**
- G1 is built by C1, with C2 to C5 forced by it.

**Findings:** none. Every change traces to G1 or is forced by a change that does, and G1 is built. `lint-pattern` is left unchanged, which matches the entry naming only the comment lint. No `Pattern-Departure:` trailer is needed.

Tool calls: none. The inherited context held the decision record, the commit log, every diff piece and the changed code.
