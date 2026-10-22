---
format: 5
approved: true
language: go
---

# Plan: quarry-csharp

LoomYard takes quarry `v0.4.0` and plans, reviews and implements with C# glyphs under `language: csharp`, with every Go plan behaving as today.
The cards raise quarry and set `Lang` on every declaration, put each ref's alphabet choice in one `internal/planparser` helper and route every parse through it, gate the Go-toolchain checks on `go`, add the language and namespace-pair refusals, read quarry's multi-alphabet answers, and teach handles, Renames and re-sign arrows the C# alphabet, ending with the package docs, PATTERN and the planner contracts.

## Card Index

1 — quarry-v040 — raise quarry to v0.4.0 and set `Lang` on every declaration
2 — planparser-ref-alphabet — one `ParseRef` helper picks each ref's alphabet; `language: csharp` accepted
3 — planglyph-ref-alphabet — `GlyphLanguage` keeps only its bool; planglyph, plandag and the kick-start parse through `ParseRef`; C# fixture
4 — scope-guard-plan — `ScopeGuard` receives the plan
5 — go-only-checks — Go-toolchain checks run only under `language: go`
6 — plan-language-mismatch — refuse a plan whose files are the other language's
7 — rename-namespace-pair — refuse a Rename pair naming a C# namespace
8 — multi-alphabet-answers — cross-alphabet ambiguity and `.go`-only build-constraint partitioning
9 — csharp-create-handles — bind C# Create handles with `Owner`
10 — csharp-rename-derivation — derive a C# Rename's declaration
11 — csharp-resign-arrows — accept and check C# re-sign arrows before the resolve
12 — resign-post-batch — rewrite a recorded C# re-sign plan-wide, or refuse it as not done
13 — batcher-resign-boundary — split a C# re-sign from a later use of its member
14 — package-docs-pattern — package docs and the PATTERN entry
15 — contracts-csharp — plan spec, planner stencil and Plan-Review rubric

## Shared Decisions

- `planparser.ParseRef(plan, ref)` is the only alphabet choice for a plan ref or a `plan:` handle's body; nothing outside `internal/planparser` passes a `glyph.Language` to a parse, and `Plan.GlyphLanguage` answers only whether the plan is glyph-enabled.
- `Plan.GoToolchain` is the one test for the Go-toolchain checks.
- The C# fixture lives at `internal/planglyph/testdata/glyphchain/csharp/Renderer.cs` in namespace `Glyphchain.Sharp`, which no Go unit of the fixture spells, and `TestGlyphChain_CSharp` is the one table its plan-gate rows extend, bar the re-sign verdicts, which extend `TestGlyphChain_ResignScenarios` with a `language` field.
- `nameDeclarations` is the package's one `quarry.Name` site from card 11 on; `resolveTargets` stays the one resolve site.

## PATTERN changes

- `PATTERN-glyph-conversion-chokepoint` — changes: Converting glyphs and paths: only via `glyph`; a plan ref's or handle's alphabet only via `planparser`.
