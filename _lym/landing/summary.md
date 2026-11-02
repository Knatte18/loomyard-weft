# Load base skills in the landing fixer, conflict and sampler seats

The landing merge fixer was told to load only the hub's language skills.
It edits code and tests, so it now loads `scribe:prose`, `scribe:code-quality` and `scribe:testing` before them, as the other code-writing seats do.

## Changes

- `internal/landingshed/fixer.go`: `FixerTable` builds the seat's `Skills` as `fixerSkills` followed by `in.LanguageSkills`, through `languageconfig.AppendSkills`.
  `fixerSkills` holds the three base skills.
- `internal/mergeresolve/spec.go`: `conflictSkills` gains `scribe:testing`, so a conflict session resolving a test file loads a testing skill.
  The package doc in `internal/mergeresolve/doc.go` is updated to match.
- `internal/samplerengine/spec.go`: the sampler session writes code and tests, so its `skills` gain `scribe:code-quality` and `scribe:testing`.
  It keeps `scribe:conversation` because it talks with the operator.

Tests assert the new skill lists in `internal/landingshed/fixer_test.go`, `internal/mergeresolve/mergeresolve_test.go`, `internal/samplerengine/spec_test.go`, and the two `internal/landingshed` integration tests that read the conflict session's skills.

## Seats checked for the same gap

Already complete, unchanged: `darnengine` (`darnSkills`), `websterengine` (`roleSkills`, on every seat that uses it), and the drift seats in `driftengine/seats.go`.

Not code writers, unchanged: `landingshed` PR reviewer (`prReviewerSkills`), `landingshed` change description (`describeSkills`), `landingshed` fix review (`fixreview.go`, a read-only judge that loads the language skills only), `treadleengine` judge and targeting, `orchcli` orch, `loomcli` driver, and the `sheddriver` tackler.

## For the reviewer

- Decide whether the sampler change belongs here.
  It was not named in the task; it was found by the same-family check.
- Confirm the `fixreview.go` fix review should stay without base skills.
  It judges a fixer's diff and writes only a report.
- Confirm `burlerengine` and `frictionengine`, which carry skill lists in their own specs, need no change.
  The `grep` that found the seats above did not inspect them individually.
- No integration or tiered gate run was recorded for this change.
