---
round: 1
focus:
  - "Shared Decision on `planparser.ParseRef`: grep the diff for any `glyph.Language` passed to a parse outside `internal/planparser`, and for `GlyphLanguage` callers that use more than its bool. Settle whether every plan ref and `plan:` handle body goes through `ParseRef`."
  - "Shared Decision on `Plan.GoToolchain`: check that every Go-toolchain check (cards 4, 5) is gated on it, and that Go plans behave exactly as before."
  - "Cards 6 and 7 refusals: check each refusal's trigger and message against the card, and that a Go plan with Go files and a Rename pair without a C# namespace are not refused."
  - "Cards 9 to 13 C# handles, Rename derivation and re-sign arrows: check `nameDeclarations` is the package's one `quarry.Name` site and `resolveTargets` the one resolve site, and that a re-sign arrow is checked before the resolve."
  - "Card 8 multi-alphabet answers: check cross-alphabet ambiguity and the `.go`-only build-constraint partitioning against the card, including the case where a C# file shares a name with a Go one."
  - "Per-card mechanical checks: for each card's groups, confirm the type-specific checks in the plan spec ran against every group's own targets, and that `assert-no-callers` holds for any Delete group."
  - "Declared PATTERN change: confirm the `PATTERN-glyph-conversion-chokepoint` entry lands with its declared text in the diff of `PATTERN.md` or `pattern/`."
  - "Test economy: check the C# fixture rows extend `TestGlyphChain_CSharp` and `TestGlyphChain_ResignScenarios` rather than duplicating existing rows, and that each added behavior has coverage."
  - "Commit messages: read `git log` over the review range for `Pattern-Departure:` trailers and judge each against the entry it names."
---

Focus follows the plan's Shared Decisions and PATTERN changes, then the rubric's `Also flag` dimensions.
