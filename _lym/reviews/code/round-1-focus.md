---
round: 1
focus:
  - "internal/commentlint/diff.go, lint.go: with workingTree true and a base other than HEAD, does `git diff <base>` (no HEAD argument) list every file whose working tree differs, rename detection and the untracked-file addition included, and does readChange read the matching base side for each? Settle whether a path deleted in the working tree or staged-only, renamed, or untracked-and-ignored is handled the way the old empty-base path handled it."
  - "internal/commentlint/lint.go Lint: an empty base becomes HEAD. Do the callers that passed an empty base (internal/gatecli/failurereport.go, the gate's comments lint step, lint-comments without --base) behave exactly as before, with no change to the findings they report?"
  - "internal/loomshed/verifygate.go: the verify gate checks the committed tree. Does LintCommitRange keep the old base-to-HEAD behaviour exactly, and is any other caller of commentlint.Lint left that should have moved to LintCommitRange (grep every caller)?"
  - "Docs and claims the diff touches: internal/loomcli/lintcomments.go doc and Long help, internal/commentlint/doc.go, docs/overview.md, pattern/PATTERN-gate-slot.md, and the other comments that name commentlint.Lint (internal/gatecli/doc.go, the loomcli file header). Does each statement match the new behaviour, and does any text left unchanged still describe the old HEAD-only reading?"
  - "Tests: TestLint_OverARepository and TestLintCommentsCmd_OverARepository. Do they assert the working-tree behaviour with independent literal expectations, and does anything pin LintCommitRange ignoring the working tree? The mutation report lists a survivor at internal/loomcli/lintcomments.go:70 (the --base flag description); judge whether any test could reasonably assert it. Check that no assertion at the review base was loosened."
  - "Commit message trailer: Root-Cause names that commentlint read the new side from HEAD whenever a base was told. Does the diff remove that defect? Check the summary's claims (callers behave as before, lint-pattern unchanged, merge danger call and blast radius) against the diff."
---

The diff is one commit that splits `commentlint.Lint` into a working-tree form and `LintCommitRange`.
The answer key is the board entry alone, whose first option, linting the working tree, the change takes.
