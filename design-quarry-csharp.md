# [.NET 006] Quarry: a C# glyph alphabet

`quarry-csharp` · Note · feature, quarry, dotnet

## Seen (orch, 2026-10-20)

Quarry links only the Go tree-sitter grammar (`github.com/tree-sitter/tree-sitter-go` in `go.mod`), so its glyph alphabet covers Go alone.
The planner takes every glyph spelling from `lyx quarry`, and Plan-Review checks the glyphs a card names, so a C# hub such as NORCE Models gets no glyphs to plan with.

## Idea

Extend quarry with a C# alphabet: link the C# tree-sitter grammar and define C#'s glyph shapes (namespace, type, member, overloads, partial classes, nested types).
Choose the alphabet by file extension, so a mixed repository gets both; the `language` module names skills, verify tiers and rename tools but no language, so it cannot pick the alphabet.
