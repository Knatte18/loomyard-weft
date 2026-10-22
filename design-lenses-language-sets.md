# [.NET 007] Language-specific lens sets seeded per hub

`lenses-language-sets` · Task · enhancement, burler, config, dotnet · running · Worktree-Create

## Seen (orch, 2026-10-20)

Each hub has its own `lenses.yaml`, seeded from the one embedded template in `internal/lenses`, and a hub can add its own lenses and fans.
So a new C# hub such as NORCE Models starts with the same lenses as this Go repository.
`pattern-fit` and `code-pattern-fit` assume a `PATTERN.md`, and no lens speaks to C# or to numerical simulation code.

## Seen (operator, 2026-10-20)

NORCE Models states much of its internal logic in its `CLAUDE.md`, which may later move into PATTERN material.
A lens that checks a hub's own rules has to read them where the hub keeps them.

## Idea

Ship language-specific lens sets beside the shared template, and let the hub pick the set that seeds its `lenses.yaml`; the `language` module names no language today, so the pick needs a key of its own.
For C#, for example: nullable and `IDisposable` handling, numerical stability and units.

## Seen (orch, 2026-10-22)

`lenses.yaml` is sparse: a hub's file already adds lenses and fans and overrides a template lens by name.
A template lens cannot be removed: a deleted one resolves to the template's value at the next load, so today it can only be dropped from the fans that name it.

## Operator decision (2026-10-22)

- A shared base, the lenses that are the same for every language, plus a predefined language layer per language (C#, Go, Python and so on) that replaces the few lenses that speak to the language itself.
- Each repository can still add its own lenses, and remove lenses it does not want.

## Orch recommendation, not a decision

- The layers ship as data beside the base template, such as one small file per language, so lyx carries no language logic in code.
- The hub picks its layer with one key in its `lenses.yaml`, such as `set: csharp`.
- Removal is a list in the hub's `lenses.yaml`, such as `remove: [<lens>]`.
- Candidates for the language layer: `pattern-fit` and `code-pattern-fit` (reading the repository's rules document wherever the hub keeps it, not `PATTERN.md` by name), `error-handling`, `concurrency`, `test-gaps`, and for C# a new lens on numerical stability and units.
