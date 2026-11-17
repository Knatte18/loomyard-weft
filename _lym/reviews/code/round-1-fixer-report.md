# Fixer-report, code review round 1

## Changes

- **F1** (MEDIUM, fixed as proposed): `code-scope` in `internal/lenses/template.yaml` now lists the `### D<n>` headings of `## Decisions` where a plan is the answer key, the board entry's goal points under `## Goal` where a decision record alone is, and the `Added after Discussion` entries in both.
  `_lym/landing/summary.md`'s opening paragraph describes the same split.
  Commit `7e629235`; the summary sits under `_lym`, so it is edited but not committed.
- **F2** (MEDIUM, fixed as proposed): `docs/dotnet-hub.md`'s `review-code` fan example gains `code-scope` between `code-tests` and `models-rules`.
  Commit `53c3d17e`.
- **F3** (LOW, fixed as proposed): the `code-scope` row is gone from the lens-phrase map in `internal/lenses/config_test.go`, and the "pins the lens's two finding sentences" line is gone from `_lym/landing/summary.md`.
  Commit `62fd56ba`.

`lym gate test ./internal/lenses` passed: 41 targets, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
