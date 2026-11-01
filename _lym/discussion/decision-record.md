## Goal

```markdown
## Seen (orch, 2026-11-01)

`contracts/specs/loom-plan-spec.md` is 112 KB, about 28k tokens, so no agent can read it whole and every reader slices it.
Its "Validation checks" section alone is about 54 KB, half the file: a catalogue of every check `internal/planparser` runs.
The next largest sections are "Plan: handles", "The shape classifier", "Card fields and order", "verify model" and "Card types", each 4 to 7 KB.
The Plan writer, Plan-Review, Code-Review and Webster's fixers all point at this one file.

## Idea (operator, 2026-11-01)

- Shorten the spec: cut what restates code, history ("Batch is gone"), rationale a reader does not act on, and anything the gate already reports with its fix.
- Split it into sub-files by reader and by need, so each role reads only the parts it uses, for example the card format a writer needs, the validation-check catalogue read only when a check refuses, and the joined-plan and part rules.
- The validation-check catalogue in particular shrinks or moves out, since `lym loom validate-plan` names each failing check; a reader needs a check's entry only when it fails.
- Every stencil, rubric and Go reference to the spec (`specs.go`, `planparser`, `websterengine`, the loom rubrics) follows the new files.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
