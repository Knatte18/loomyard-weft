# Fixer-report, round 1

## Changes

- F1 (LOW): fixed as proposed, commit `7c4ae6d5`.
  `TestShowFile_ReadsCommittedBytes` gained `ShortSHA`, `AnnotatedTag` and `DirectoryPathIsNil` rows, and an annotated tag `v1` created before the table.
  The test's doc comment now names the new revisions.
- F2 (LOW): fixed as proposed, commit `76d17b49`.
  `TestRevParse_HeadReadFromFilesMatchesGit` gained a `SymlinkedWorktree` row; its link is created with `os.Symlink` under `t.TempDir()`.

A gate then flagged the F1 doc comment as wrapped at a fixed column.
It is now two sentences, one per line, each under 120 characters.

`lym gate test --tier 1 ./internal/gitkit ./internal/fabricengine` passed with 0 failures.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
