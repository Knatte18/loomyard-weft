## Goal

```markdown
## Seen

2026-11-19: Finalize of `hardener-campaign` stopped on an untracked zero-byte `internal/hardenshed/.lock`, created 18:39 during the run's gate, held by no process.
`internal/hardenshed`'s `round.go` and `live.go` take their lock through `lock.TrackedFileLockPath(path, ".lock")`; a test that passes a relative or empty record path makes that lock land in the package directory, which is the test's working directory.
The orch removed the file by hand so Finalize could go on; the next gate run recreates it.

## Idea

Find the hardenshed test that writes a record outside `t.TempDir()` and point it there.
A gate check that fails on any new untracked file in the source tree after a test run would catch the whole class.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): No hardenshed test fix; stray-file gate check instead

Decision: The hardenshed test fix is dropped: the stray .lock did not reproduce on 7346d6a8 (every hardenshed test builds its paths under t.TempDir). The run builds only the gate check that fails a tier run leaving new untracked, non-ignored files.

Rationale: The file came from the campaign's pre-squash tree or another process; the gate check catches the whole class wherever it comes from.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - copies-run-drops-stray-failure-from-report (class design, severity MEDIUM, open in rounds 1)
  - rerun-treats-first-attempt-stray-files-as-present (class design, severity MEDIUM, open in rounds 1)
  - stray-test-windows-runtime-skip (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
