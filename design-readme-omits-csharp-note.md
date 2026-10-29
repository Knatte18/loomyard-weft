# Revise the board README rendering

`readme-omits-csharp-note` · Note · enhancement, board

## Seen (operator, 2026-10-29)

The operator wants the README rendering revised.
The trigger: the note `csharp-sandbox-hub` was a member of the project `dotnet`, so it rendered only on `project-dotnet.md` and left the README's type groups, while the README listed the project as one line with its brief, no member count and no titles.
The note looked gone, and a project with members looked as empty as one without.

## Design (operator, 2026-10-29)

- Show each project's members, or at least their count, under its line in the README, so no entry disappears from the README's view.
- The notes tables have a `P` column that holds the priority, `L` for low, while `P` is also the handle letter of a project: rename the column so the two no longer share a letter.
- A project page carries a stray line reading `grouping` under its brief: remove it.

## Open

- The column's new name.
- Whether the README shows the members' titles or only their count.
