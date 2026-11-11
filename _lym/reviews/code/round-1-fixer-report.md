# Fixer report — round 1

## Changes

All three findings fixed as proposed; each is its own commit. `lym gate test ./internal/boardcli ./internal/boardengine` passed (tier0, no failures).

- **F1** (`4ac3ace1`): `plugins/ly/skills/board/SKILL.md` — after the Sketch paragraph of "Promote a note to a task", added the sentence that a Sketch's dependencies go in the promotion, with the `{"slug":"my-sketch","depends_on":["other-task"]}` example.
- **F2** (`67207c0d`): added a `promote names the sibling` case to `stepSiblingWarning` in `internal/boardcli/cli_test.go`, asserting the exact notice line, and a promote row to `TestWriteSiblings` in `internal/boardengine/siblings_test.go`, asserting the sibling and the stored `depends_on`.
- **F3** (`cac5826c`):
  - `internal/boardengine/sketch_test.go`: rows `DependsOnEqualToATasksOwnIsANoOp` and `EmptyDependsOnOnATaskWithDependenciesIsRefused` in `TestDiscussedPromotion`.
  - `internal/boardcli/verbs_test.go`: a null `depends_on` case in `stepPromote`, and a `depends_on not strings` row in `stepPromote_Refusals`.
  - Gate repair: the first null case seeded a note with `depends_on`, which the store refuses ("only a task can depend on another entry").
    A note never holds dependencies, so the case now promotes a note with none and asserts the task has none; the review's "clears a note's deps" wording does not apply.

## Self-check

All four forks returned a report; each said "no findings".

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
