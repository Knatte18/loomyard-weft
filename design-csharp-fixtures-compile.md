# [.NET] A test compiles the C# test fixtures, skipped without dotnet

`csharp-fixtures-compile` · Task · enhancement, quarry, dotnet, testkit · running · Worktree-Create

## Seen

`quarry-csharp` added C# test fixtures, such as `internal/planglyph/testdata/glyphchain/csharp/Renderer.cs`.
Quarry parses them with tree-sitter, which tolerates syntax errors, so nothing proves a fixture is valid C#: a broken fixture would still pass the glyph tests.

## Idea

A test compiles every `.cs` fixture under `testdata/` with `dotnet build` in a throwaway project, and skips when `dotnet` is not on `PATH`.
It belongs in a gate tier that the machines with .NET run, so a Go-only machine stays green.
