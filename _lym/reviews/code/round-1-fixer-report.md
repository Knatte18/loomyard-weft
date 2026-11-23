---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

# Fixer-report, round 1

F3 is a `decision`-class finding held by the parent, so this report leaves it out.

## Changes

Commit `5df1cca42` holds F1 and F2, which both change `view_test.go`.

- F1: `ViewProject` in `internal/boardengine/view.go` gains `Done bool` with JSON key `done`.
  The `add` closure in `viewProjects` takes it as a parameter: false from the open walk, true from the done loop.
  `projectSelect` in `internal/boardengine/board.html` skips a project whose `done` is true, so the create and edit forms no longer offer a project every write refuses.
- F2: `viewFixture` in `internal/boardengine/view_test.go` gains done project P5 `epsilon` inside done project `gamma`, and its doc comment is updated.
  The projects test expects P3 with `Done: true`, then P5 with `Parent: "P3"` and `Done: true`.
- `lym gate test ./internal/boardengine` passed on tier0 with 0 failures.

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
