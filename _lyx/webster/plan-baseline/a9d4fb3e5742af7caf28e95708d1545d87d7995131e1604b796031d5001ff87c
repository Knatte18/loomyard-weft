# Card 9 — resign-arrow-parse

**Create:**
- `plan:internal/planparser#CardResign` -> `type CardResign struct`

**Edit:**
- `internal/planparser#Card`
- `internal/planparser#TargetGroup`
- `internal/planparser/parse.go`
- `internal/planparser/validate.go`
- `internal/planparser/shape.go`
- `internal/planparser/doc.go`
- `internal/planparser/parse_test.go`
- `internal/planparser/validate_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Give `**Edit:**` bullets D5's re-sign marker, `` `<glyph>` -> `<new declaration head>` ``, parsed by `planparser` alone (`PATTERN-sole-parsers`), plus the pure blocking check `resign-not-member`.

- `CardResign` carries one arrow bullet: `Target`, the left token verbatim, and `Decl`, the head verbatim.
  `TargetGroup.Resigns` holds an `Edit` group's own arrow bullets and `Card.Resigns` their flat union, in body order.
- `parse.go` gives `**Edit:**` its own field parser beside `parseCreateField`: a bullet matching `moveLineRe` contributes a `CardResign` and its left token to the group's `Refs`, so the member stays an ordinary Edit target for every other consumer; any other bullet parses as today.
  A member glyph passes normalization verbatim, so `Target` equals its `Refs` entry.
- `validate.go` adds `resign-not-member` (header list included), run after `checkHandleMalformed`, under a glyph-enabled `language:` only: an arrow whose target is not a member glyph, classified through a new ledger gate in `shape.go` (a path, bare symbol or `plan:` handle is the finding; a glyph is kept, and then a self glyph is the finding, while a glyph `glyph.Parse` rejects is left to `glyph-malformed`).
  Its detail names the target's shape and the way forward: put the arrow on the member glyph whose signature changes, since a file, a package or a handle has no prior signature to re-sign.
- `doc.go`'s type model names `Resigns`.

`parse_test.go` covers an arrow bullet landing in `Resigns`, `Refs` and `Targets`, and a plain bullet carrying no resign; `validate_test.go` covers the finding for a file path, a file self glyph, a package self glyph and a handle, and a member glyph arrow validating clean.

The spec's Card fields section describes the arrow on `**Edit:**` bullets (marks a signature change and states the new head, parsed like a `Create` head), and gains the row, renumbered with every `row N` cross-reference fixed.

**ImpactSummary:** Adds a parsed field and one pure blocking check; an Edit bullet without the arrow parses exactly as before, and an arrow bullet that used to fail as a malformed glyph now parses.
