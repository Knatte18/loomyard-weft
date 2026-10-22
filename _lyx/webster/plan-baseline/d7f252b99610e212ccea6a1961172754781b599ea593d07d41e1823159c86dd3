# Card 14 — package-docs-pattern

**Prosa:**
- `internal/planparser/doc.go`
- `internal/planglyph/doc.go`
- `internal/batcher/doc.go`
- `internal/websterengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:** Write the built rules into the package docs and the PATTERN entry (`PATTERN-documentation-lifecycle`, `PATTERN-glyph-conversion-chokepoint`).

- `internal/planparser/doc.go`: the `language:` section names `csharp` beside `go` and `none`; `ParseRef` as the one alphabet choice and its three `csharp` rules; canonicalization through the Go unit alphabet under both glyph languages; `PathEntryPath` and its two callers; D3's bound for slash-free self glyphs under `csharp`; `GoToolchain` and the checks it gates; `plan-language-mismatch` and `rename-namespace-pair`.
- `internal/planglyph/doc.go`: C# handle binding with `Owner` and the interface-qualifier rule; the C# Rename derivation; re-sign naming before the resolve, the minted-id acceptance rule, `resign-overload-collision`, and `RebindResigns` with `resign-not-done`; the cross-alphabet `glyph-ambiguous` detail; build-constraint partitioning over `.go` candidate sets only; `nameDeclarations` as the one `quarry.Name` site.
- `internal/batcher/doc.go`: the C# re-sign boundary beside the cover rule.
- `internal/websterengine/doc.go`: the post-batch pass's re-sign rebind, its restamp, and the re-read plan the scope guard and drift detection get.
- `PATTERN.md`: the entry line becomes exactly:

```markdown
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only via `glyph`; a plan ref's or handle's alphabet only via `planparser`. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains the alphabet rule, that `planparser.ParseRef` alone picks a plan ref's or handle's alphabet from the plan's `language:`, and the rule that path self glyphs use the Go unit alphabet under every glyph-enabled language, with `planparser.PathEntryPath` the one Go-alphabet path reading of a canonicalized path entry.

Every file keeps one sentence per line.
