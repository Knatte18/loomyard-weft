# Revise the board README rendering

`readme-omits-csharp-note` · Note · enhancement, board

## Seen (operator, 2026-10-29)

The operator wants the README rendering revised.
The trigger: the note `csharp-sandbox-hub` was a member of the project `dotnet`, so it rendered only on `project-dotnet.md` and left the README's type groups, while the README listed the project as one line with its brief, no member count and no titles.
The note looked gone, and a project with members looked as empty as one without.

Other unclear spots seen in the same read:

- The notes tables have a `P` column that holds the priority, `L` for low, while `P` is also the handle letter of a project.
- A project page carries a stray line reading `grouping` under its brief.

## Idea

Show each project's members, or at least their count, under its line in the README, so no entry disappears from the README's view, and settle the rest of the revision with the operator.
