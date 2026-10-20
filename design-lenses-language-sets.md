# [.NET 007] Language-specific lens sets seeded per hub

`lenses-language-sets` · Note · enhancement, burler, config, dotnet

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
