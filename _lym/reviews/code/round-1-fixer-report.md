# Fixer-report, round 1

## Changes

Commit `298d0702`, `internal/boardengine/render_test.go`; `lym gate test ./internal/boardengine` passed.

- **F1, fixed as proposed.** Fixture task T2, which has an empty recipe, got `Profile: "a|b"`, and its golden Recipe cell became `tapestry · a\|b`. The row now pins the default-recipe branch together with the profile and its pipe escaping. The `readmeFixture` doc comment names the two profile cases. No new test.
- **F2, fixed as proposed.** The `TestRenderReadmeGolden` doc comment now says `tapestry` for an empty recipe, followed by ` · ` and the pipe-escaped profile when the task names one.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
