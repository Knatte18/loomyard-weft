# Round 1 fixer-report

## Changes

- F1 (LOW, consistency): `docs/research/twill/twill-discussion-bench/deviations.md` now cites `report.md` on lines 1 and 5.
  Fixed as proposed, in two commits: the first `replace_all` matched only the backticked name on line 1, and the self-check forks found line 5 still unquoted.
  A repo-wide search of `docs` and `internal` for `twill-discussion-bench.md` now finds nothing.
- F2 (LOW, consistency): the step-0 `cp` in `docs/research/glyph/glyph-audit-2026-10-15/COMMANDS.md` line 13 now copies from `docs/research/glyph/glyph-audit-2026-10-13/`, agreeing with line 101.
  Fixed as proposed, first option.
- F3 (NIT, design): `docsLinkIsResearchData` in `internal/lymcwd/docslink_test.go` now requires `len(parts) > 5` and looks for `data` in `parts[4:len(parts)-1]`, so a `data/` folder directly under a topic is scanned.
  The research-data subtest gained the row `docs/research/topic/data/x.md`, expected as a break, and the predicate's doc comment states the topic-level case.
  Fixed as proposed, the review's first option.
  No existing `data/` folder sits directly under a topic, so no file is newly scanned.

`lym gate test ./internal/lymcwd` passed before the commits: tier0, 148 passed, 0 failed, 2 skipped.
The later line-5 commit touches a markdown file only.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: line 5 of `deviations.md` still named `twill-discussion-bench.md`. Fixed as proposed.
- fix-affected-set, MEDIUM: same line 5 miss, merged with the fix-consistency finding. Fixed as proposed.
- fix-proposal, LOW: a series-level `data/` folder (`docs/research/topic/series/data/x.md`) is still skipped. Deferred, see below.

## Disputed

None.

## Deferred

- fix-proposal LOW, series-level `data/` folder: the predicate cannot tell a series folder from a study folder without reading the tree, so closing this boundary needs a tree-aware predicate or an operator decision on whether a series-level skip is intended.
  No such folder exists today, and the review's F3 asked about the topic level only.
