---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/lintcomments_integration_test.go:22
    summary: The test doc says every step asserts lint-comments and lint-pattern agree, but the new uncommitted-edit step runs lint-comments alone, and lint-pattern --base no longer reads the same tree.
    fix: "In the doc comment of TestLintCommentsCmd_OverARepository, replace the sentence \"At each step it asserts lint-comments and lint-pattern agree on the same tree.\" with one saying the steps up to the base commit assert both verbs agree, and the last step checks that lint-comments --base also reports an uncommitted edit."
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md#What changed
    summary: The summary claims the empty-base callers behave as before, but an untracked .go file that HEAD still holds now has only its changed lines linted, where every line used to count.
    fix: "In _lym/landing/summary.md under ## What changed, after \"Its callers without a base, the gate's `comments` lint step and `lint-comments` without `--base`, behave as before.\", add one sentence: an untracked `.go` file that the base still holds, as after `git rm --cached`, is now compared with its base copy, so only its changed lines count."
    origin: handler
---

### [LOW] Test doc claims both verbs agree at every step

**Location:** `internal/loomcli/lintcomments_integration_test.go:22`
**Issue:** The doc comment reads "At each step it asserts lint-comments and lint-pattern agree on the same tree."
The step this diff adds, "lint-comments --base lints uncommitted edits too", calls only `lint("lint-comments", "--base", base)`.
`lint-pattern --base` still compares `--base` with HEAD (`docs/overview.md`, the `lint-pattern` line), so on that step the two verbs no longer read the same tree, and the doc overstates what the test asserts.

### [LOW] Summary's "behave as before" claim is not exact

**Location:** `_lym/landing/summary.md#What changed`
**Issue:** For an empty base, `changedGoFiles` now probes each untracked `.go` path with `cat-file -e HEAD:<path>` (`internal/commentlint/diff.go:49-56`) and sets `basePath` when HEAD holds it.
Before the diff, every untracked path had an empty `basePath`, so a file removed from the index but kept at HEAD had every line linted as new.
`gatecli`'s `runLints` (`internal/gatecli/failurereport.go:118`) and `lint-comments` without `--base` now report fewer findings for that file.
The new result is the more correct one, but the summary's claim that these callers "behave as before" does not quite hold.

## Rejected

- A redundant probe cost: `cat-file -e` now runs once per untracked `.go` file for the empty-base callers too. That is one local git process per file, needed for correctness, and no defect.
- `LintCommitRange` given an empty base would fail in git. Its one caller guards with `lintBase != ""` (`internal/loomshed/verifygate.go:145`), and the package is internal, so the caller is trusted.
- The mutation survivor at `internal/loomcli/lintcomments.go:70` is the `--base` flag's help text, static prose that the rubric exempts from coverage. No test could reasonably assert it.

Focus entries settled:
- Working-tree diff against a non-HEAD base: `git diff <base>` lists staged and unstaged changes and staged renames. Deletions are skipped as before, and ignored files stay out through `--exclude-standard`. `readChange` reads the base side at `baseRevision` for each file.
- Empty-base callers: the git arguments are unchanged; the one difference is F2.
- `LintCommitRange` is the old base-to-HEAD form exactly, and the verify gate is its only caller. The other callers of `commentlint.Lint` (`gatecli` `runLints`, `lint-comments`) correctly stay on `Lint`.
- Docs: `internal/gatecli/doc.go`, the loomcli file header, the package doc, `docs/overview.md` and `pattern/PATTERN-gate-slot.md` all match the new behaviour. The one stale text is the test doc in F1.
- Tests use literal expectations, and the verify-gate test pins that `LintCommitRange` ignores an untracked file. No assertion that exists at the review base was loosened.
- The `Root-Cause:` trailer on `e1f36ce73` names the defect the diff removes. The merge danger call, two-way with blast radius lint, holds: no record format, config meaning or outward push changes.
