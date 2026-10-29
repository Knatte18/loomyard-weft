# The board README omits the note csharp-sandbox-hub

`readme-omits-csharp-note` · Note · bug, board

## Seen (operator, 2026-10-29)

The note `csharp-sandbox-hub` does not appear in the rendered board README, though `lym board list` shows it as a note with the label `dotnet`.
The project `dotnet` listed it in `order` while the note had no `project` field.
Setting `project` to `dotnet` made the link two-sided, and the README was regenerated in the same second as that write, but the operator still did not see the note.

## Idea

Find which entries the README renderer drops, and why: an entry named in a project's `order` without pointing back, or a note with a module label of a project, are the first suspects.
A project whose members are all missing should not render as an empty heading, and no entry should vanish from the README.
