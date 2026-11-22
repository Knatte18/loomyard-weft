## Goal

```markdown
## Goal (operator, 2026-11-22)

Make the rendered board README less cluttered, tried out with the operator in a sampler session:

- **Beam marker**: mark each project that has a beam, in its heading and in any project list.
- **No empty sections**: render no section, heading or table that holds no entry, such as a project's empty Tasks and Ready.
- **No Notes level**: drop the Notes heading above the type groups; Features, Bugs, Enhancements and the other type groups become sections of their own.

The session shows the operator renderings of the current board and keeps what the operator picks.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-22): No section meaning lines

Decision: The board files render no one-line meaning under a section heading: Tasks, Running, Ready, the dependency layers, Independent, Draft, Notes, Done and Own entries lose theirs.

Rationale: The operator judged the explanations clutter.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
