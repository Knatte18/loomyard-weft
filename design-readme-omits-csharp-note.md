# Revise the board README rendering

`readme-omits-csharp-note` · Task · enhancement, board · running · Drift-Verify

## Seen (operator, 2026-10-29)

The operator wants the README rendering revised.
The trigger: the note `csharp-sandbox-hub` was a member of the project `dotnet`, so it rendered only on `project-dotnet.md` and left the README's type groups, while the README listed the project as one line with its brief, no member count and no titles.
The note looked gone, and a project with members looked as empty as one without.

## Design (operator, 2026-10-30)

- Each project has its own section in the README, at the very top, so no entry disappears from the README's view.
  A section holds the project's brief and its members, each with its handle and title, laid out as the README lays out the entries of a type group.
- The prime, the root project, is a project too: its section comes below the sections of the other projects.
- The notes tables have a `P` column that holds the priority, `L` for low, while `P` is also the handle letter of a project: rename that column to `Prio`, so the two no longer share a letter.
- A project page carries a stray line reading `grouping` under its brief: remove it.
