## Goal

```markdown
## Seen

Only a task carries `depends_on` (`lym board set-deps` and `upsert` refuse it on a note; `promote` takes it at the promotion).
A note that must wait for a running task, such as `harden-fabric-undriven-cases` waiting for `harden-fabric-single-repo-verbs`, shows nothing of that on the board,
so the operator keeps asking whether it can start.

## Wanted

Operator, 2026-11-23:

- `upsert` and `set-deps` accept `depends_on` on a note.
- `lym board list --text` and the README show a note's dependencies as they show a task's.
- `promote` keeps a note's `depends_on`; one given in the promote payload replaces it.
- `prune` strips a done entry from a note's `depends_on` as from a task's.
- The `ly:board` and `ly:board-policy` skills say a note may carry `depends_on`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): What an entry may depend on

Decision: A task or a note may depend on any other entry: a task, a note or a project. A cycle is refused, as for tasks today. A task that depends on a note cannot start until that note has become a task and landed, or is removed, and the board shows it waiting. Removing an entry strips it from every other entry's depends_on, as prune does for a done entry.

Rationale: Operator, 2026-11-23: a note may depend on a note, such as one idea building on another.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
