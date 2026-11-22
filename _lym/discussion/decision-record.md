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

### Added after Discussion (operator, 2026-11-22): No Tasks level

Decision: The board files render no Tasks heading: Running, Ready, each dependency layer, Independent and Draft become sections of their own, at the level Tasks held, beside the note type sections and Done.

Rationale: The operator judged the Tasks heading clutter, as the Notes heading was.

### Added after Discussion (operator, 2026-11-22): No Tasks level

Decision: The board files render no Tasks heading: Running, Ready, each dependency layer, Independent and Draft are sections of their own, at the level of the note type sections and Done.

Rationale: The operator judged the Tasks heading clutter, as the Notes heading was.

### Added after Discussion (operator, 2026-11-22): No collapsible project sections

Decision: README.md renders each project as a plain level-two heading linking its file, not a details block.

Rationale: The operator found the collapse useless: a re-render forgets it.

### Added after Discussion (operator, 2026-11-22): Beam marker reads BEAM after the title

Decision: A beam project's heading, in README.md and in every project file, and its Done row end in ' · BEAM', after the title and the member count, with no emoji.

Rationale: The operator found a marker before the title unreadable and an emoji meaningless.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
