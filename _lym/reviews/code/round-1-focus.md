---
round: 1
focus:
  - "Old-path sweep: grep the tree (code comments, docs, README.md, templates, scripts, reports) for every pre-move path under docs/research/ that `git diff --name-status -M e0e6de2c..HEAD` shows as renamed. Settle whether any reference to an old path survives outside git history, which the decision record's closing grep demands."
  - "Study scripts under docs/research/: settle whether every script that located the repository root by `../../..` now uses a depth-independent means, and whether any other relative path in a script (data dirs, sibling files, sourced helpers) still assumes the old depth."
  - "Relative links inside moved reports and the new docs/research/README.md: settle whether each link resolves from the file's new location and whether the index lists every study folder and lone report with a line on what it measured."
  - "internal/lymcwd/docslink_test.go `docsLinkIsResearchData`: settle whether the `slices.Contains(parts[3:len(parts)-1], \"data\")` predicate matches only a study's data/ directory (no false skip of an ordinary doc, no missed case), and whether the changed subtest covers the depths it claims."
  - "internal/lymcwd/enforcement_test.go `vocabScanAllowlist`: settle whether the two new entries are needed by the moved notes, are scoped to exactly those files, and are justified by the diff, not a loosening of the scan."
  - "Moved design notes (reed-proposal, reed-exploration, logging-inventory, quarry-holistic-fix-log, weaving-glossary): settle whether each lands where the decision record says (docs/ or beside the design) and whether every link into and out of it was fixed."
  - "Nothing deleted: settle whether every file removed at its old path appears at a new path with content intact except for intended path and link edits, and whether any content edit beyond reference fixes (report.md renames from README.md, data files) alters a measurement report's substance."
  - "Commit message claims (README-only reports become report.md; benchmarks and crucible template references updated; docs/overview.md updated): settle whether the diff bears out each claim."
---

Round 1 focus follows the decision record's closing demand: no stale reference to a moved path, and no script, test or link left depending on the old depth.
