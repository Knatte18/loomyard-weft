# Fixer-report, webster round 1

## Changes

- F1 (MEDIUM): fixed as proposed.
  `wireLanding`'s undecodable-seed error in `internal/loomcli/sharedbootstrap.go` now says that a re-seed after Publish opened the pull request moves the run's start past it, so PR-Gate, PR-Align and Finalize find no pull request and Publish stops on it as an older open one.
  It names the remedy: restore the original `started_at` in the seed by editing the file, or close that pull request and resume with `lyx loom start`.
  `TestWireLanding_TellsLanguageSkills` asserts the `started_at` clause.
  A second commit added the "by editing the file" wording after the self-check.
- F2 (LOW): fixed as proposed.
  `internal/loomcli/doc.go` states that the landing wiring reads the run's start from the pair's seed and fails on an undecodable one, and that `lyx loom approve` and `lyx loom reject` look up the pull request created at or after that start and refuse when the seed cannot be read.
- F3 (LOW): fixed as proposed.
  `TestParentReviewAligner` has a row with a new `notesBefore` field: a counted decision note seeded before the spawn, its body changed after, wanting `VerdictEscalate` with `DecisionRecordedReason`.

`lyx gate test ./internal/loomcli` and `lyx gate test --tags integration ./internal/loomcli` pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW on F1, the "restore the original started_at" remedy gave no way to do it. Fixed with a small wording change naming the file edit and the time bound.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
