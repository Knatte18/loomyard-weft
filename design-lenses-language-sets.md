# [.NET 007] Language-specific lens sets seeded per hub

`lenses-language-sets` · Note · enhancement, burler, config, dotnet

## Seen (orch, 2026-10-20)

Each hub has its own `lenses.yaml`, seeded from the one embedded template in `internal/lenses`, and a hub can add its own lenses and fans.
So a new C# hub such as NORCE Models starts with the same lenses as this Go repository.
`pattern-fit` and `code-pattern-fit` assume a `PATTERN.md`, and no lens speaks to C# or to numerical simulation code.

## Idea

Ship language-specific lens sets beside the shared template: a hub's language, as configured in the `language` module, picks which set seeds its `lenses.yaml`.
For C#, for example: nullable and `IDisposable` handling, numerical stability and units.
