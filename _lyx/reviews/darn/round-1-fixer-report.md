## Changes

- F2: removed the gate-busy `Contains` row from the `want` list in `TestBuildDenyNotice_SentenceContent` (`internal/shuttleengine/claudeengine/settings_test.go`), fixed as proposed.
  `TestBuildSettings_MatchesGolden` still pins the wording.
  Commit `6d731073e`; `lyx gate test ./internal/shuttleengine/claudeengine` is green.

## Disputed

Nothing.

## Deferred

- F1: left in place as the finding itself proposes; the fixture change is test-only and reverting it would bring back the intermittent loop-wait failures.
  Recording it with `lyx loom decision add` or splitting it into its own task is the run's parent's call, outside this write surface.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
