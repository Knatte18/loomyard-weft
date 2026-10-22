# [.NET 006] LoomYard uses quarry C# glyphs: version raise, alphabet by extension

`quarry-csharp` · Task · feature, quarry, dotnet · running · PR-Align

## Seen (orch, 2026-10-20)

Quarry knows only Go glyphs, so LoomYard's planner and Plan-Review get no glyphs in a C# hub such as NORCE Models.

## Split (operator, 2026-10-21)

Quarry is its own repository and hub (`Knatte18/quarry`, hub `quarry-LYXHUB`).
The C# grammar and glyph alphabet are built there, in that hub's task `csharp-alphabet`.

This entry is LoomYard's side only, started once `csharp-alphabet` has landed and quarry has a new version tag:

- raise the quarry version in `go.mod`;
- pick the alphabet per file, by extension, wherever LoomYard asks quarry for glyphs;
- the planner, Plan-Review and the card kick-start get C# glyphs in a C# hub.

A first run of this entry was started too early on 2026-10-21 and abandoned at Discussion-Write.

## Ready (2026-10-22)

`csharp-alphabet` landed in quarry as `ca716c2`, and quarry tagged it `v0.4.0` (operator decision, 2026-10-22); `go.mod` pins `v0.2.1` today, so the raise also takes in `v0.3.0`'s Enclose, which LoomYard does not use yet.
The quarry commit names a library break: a declaration input with a zero `Lang` is now rejected, so every LoomYard caller must set `Lang`.
The LoomYard packages importing quarry are `internal/planglyph`, `internal/planparser` and `internal/quarrycli`.
