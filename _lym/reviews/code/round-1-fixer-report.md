# Fixer-report, round 1

All six findings are fixed as proposed, except for F3, which is fixed with an own change. The gate (`lym gate test ./tools/tokencount --tier 0,1`) is green.
Commits: `22381f05` (F1–F6), `5983ba68` (self-check) and a gate repair commit.
The gate then flagged a fixed-column-wrapped comment in the `main.go` package doc (the `-last` sentence); it now breaks at the sentence boundary, one sentence per line.
The fixes went in as one commit for the six findings, not one commit each, because the shared test files were edited together.

## Changes

- **F1** — `findLanding` (`landed.go`) returns `Landing{Missing: noParsableDescription + ": " + <parse error>}` with a nil error when the archived description does not parse, so the run is listed under `No landed commit`.
  The `TestFindLanding` row `a change description that does not parse` now expects that Landing (`parseFails`).
- **F2** — `TestRunReadsArchivedShedStatus` runs with `-last 1 -since 2026-01-01` and asserts the `active since` header with the skipped runs; `TestParseSince` (`main_test.go`) covers a valid date and the empty value.
- **F3 (fixed with my own change)** — The proposal was to diff against the empty tree or to return a Missing reason on a failed diff.
  I made `ChangedLines` run `git show --numstat --format= -m --first-parent <sha>`, which diffs a root commit against the empty tree without hard-coding the tree hash, and diffs a merge against its first parent.
  `TestCodeRepoChangedLines` (`time_integration_test.go`) covers a root commit and a regular commit.
- **F4** — `run` refuses `-last` below 1 when no slug and no `-since` are given, naming the flag; two rows added to `TestRunRefuses`.
- **F5** — both wrapper headers now say the report opens with the most expensive parts, the cost per run and the recipe comparison.
- **F6** — `TestWriteRecipes` gains a zero-line run (`0`, `n/a`), a partly unpriced run and a wholly unpriced run; the integration check of the changed-line count is the full `tapestryRow` regexp.
- **Package doc** — `main.go` notes that `-last` is at least 1 and that an unparsable description is listed under `No landed commit`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW (package doc lacks the `-last` bound and the parse reason): fixed as proposed.
- fix-consistency, LOW (parse reason is long inside the list): disputed, premise false (see Disputed).
- fix-proposal, LOW (`--diff-merges` needs git 2.31): fixed with a change; replaced by `-m --first-parent`.
- fix-proposal, LOW (parse reason could garble the list): disputed, premise false (see Disputed).
- fix-proposal, LOW (`-last` refusal note): no defect named; nothing to change.
- fix-proposal, LOW (`-since` test does not pin the `-last` override): disputed, premise false (see Disputed).
- fix-affected-set: no findings.

## Disputed

- **Parse reason garbles the list** (fix-consistency #2, fix-proposal #2): premise false. `summaryparser` quotes the first line with `%q`, so the reason holds no raw newline, and the list stays on one line. Dropping the error text would hide why the run has no landing.
- **`-since` test does not pin the `-last` override** (fix-proposal #4): premise false. With `-last 1` and the override removed, the loop stops after `full` and the header says `Skipped 2 ... done, live`; the test asserts `Skipped 3 ... done, live, solo`, so it fails without the override.

## Deferred

Nothing deferred.
