# Fixer-report — code review round 1

## Changes

- **F1 (MEDIUM)** fixed as proposed, commit `5cb5f365`.
  `internal/driftengine/config.go`: `ConfigSpec` declares `OpenMapShapes` for `capture` and `compare`, each `yamlengine.ShapeOf(stepFile{})`.
  The template comments and the `ConfigSpec` doc name the one-key form.
  `internal/driftengine/config_test.go`: `TestConfigSpec_SetNestedKeys` sets `capture.timeout_min`, `capture.env.NAME`, `compare.command` and `compare.timeout_min` through `configengine.Set`, loads with `LoadConfig`, and checks a misspelled `capture.timeoutt` is refused naming it with the file unchanged.
  The `configcli` doc sentence is generic over a declared shape and needed no change.
- **F2 (MEDIUM)** fixed as proposed, commit `e8312bbd`.
  `TestSetValues` rows for a whole entry from a flow mapping (`tiers.t0`) and a bracketed value on a new `covers []int` field of `shapedTier`; one `TestSetSparse` row for the whole-entry form.
- **F3 (LOW)** fixed as proposed, same commit.
  `shapedTier` gained an untagged `Plain`, an `omitempty` `Retries` and an unexported `secret`; a `TestSetValues` row sets the first two, and a `TestSetValues_Refusals` row names `secret` as a key the entry lacks.
  The sorted key list in the existing `envv` refusal row was updated.
- **F4 (LOW)** fixed as proposed, same commit.
  `TestSetValues_ShapedRefusalRejectsWholeCall` sends a valid shaped pair then a misspelled one and asserts the error and a nil `Merged`.
  F2, F3 and F4 share one test file and type, so they are one commit.
- **Self-check follow-ups**, commit after `e8312bbd`: `internal/driftengine/doc.go` and both template comments now say the one-key set applies once a step is declared; a `TestSetValues` row pins a plain value for a whole shaped entry.

Gate: `lym gate test ./internal/yamlengine ./internal/driftengine ./internal/gateslot ./internal/configengine ./internal/configcli` passed after the first fixes; `./internal/yamlengine ./internal/driftengine` passed after the follow-ups.

- **Gate repair**: the `shapedTier` doc comment in `internal/yamlengine/set_test.go` was wrapped at a fixed column; it is now one sentence per line.

## Self-check

- fix-consistency, MEDIUM: `driftengine/doc.go` still described only the whole-step form. Fixed as proposed.
- fix-proposal, MEDIUM: the template comment offered `capture.timeout_min=30` on the undeclared `{}` default, which the load check refuses (`capture.command: missing`). Fixed with my own change: the comments say "once declared", and `doc.go` says a first declaration sets `command` and `timeout_min` together.
- fix-code-correctness, LOW: `TestSetValues_ShapedRefusalRejectsWholeCall` passes even if validation ran per pair. Disputed, ground every fix worse (see Disputed).
- fix-code-correctness, LOW: the whole-entry rows do not reach the `Fields`/`Entries` cases of `setShapedValue`, since a `{` value takes the brace case. Fixed with my own change: a row sets `tiers.t0=5` and asserts `t0: "5"`, which only those cases produce.
- fix-affected-set: no findings.

## Disputed

- fix-code-correctness LOW on F4's test, ground `every fix worse`.
  `SetValues` returns `SetResult{}` on every error path (`internal/yamlengine/set.go:201-203` and the apply loop), so no observable output tells up-front validation from per-pair validation.
  Dropping the test would lose the pinned contract of an error and nil `Merged` for a valid-then-invalid call; no other change makes the regression observable.

## Deferred

None.
