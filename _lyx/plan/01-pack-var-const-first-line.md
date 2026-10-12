# Card 1 — pack-var-const-first-line

**Edit:**
- `internal/planglyph#appendPackAnswer`
- `internal/planindex#PackDeclaration`
- `internal/planglyph#TestKickstartPack`
- `internal/websterengine/doc.go`

**Intent:** Decision D5: a `var` or `const` declaration in the kick-start pack shows the first line of its signature only.

In `appendPackAnswer`, when a quarry symbol's `Kind` is `quarry.KindVar` or `quarry.KindConst`, the `PackDeclaration`'s `Signature` is the signature's text up to its first newline, with trailing whitespace trimmed (`strings.TrimRight` over `" \t\r"`).
Every other kind keeps quarry's signature verbatim, a multi-line function or method signature included.
`File`, `Start` and `End` are unchanged, so reading the span still shows the whole initializer.
quarry prefixes a grouped spec's signature with its keyword, so the first line reads `var name …` or `const name …` in both the ungrouped and the grouped form.

`PackDeclaration`'s doc comment states the rule: the signature is quarry's verbatim, except a `var` or `const` keeps its first line only, since its signature carries the whole initializer.

`TestKickstartPack` gains these declarations at the END of its shared `source` fixture, so the existing rows' line numbers stay put: an ungrouped `var` whose initializer spans several lines (a composite literal), a `var ( … )` group whose second spec's initializer spans several lines, an ungrouped `const` whose initializer spans several lines (e.g. `"a" +` on one line, `"b"` on the next), a `const ( … )` group one of whose specs has an initializer spanning several lines in the same way, a function `Long` whose parameter list spans several lines, and a type `T` with a method `(T) LongMethod` whose parameter list spans several lines.
Every `var` and `const` the row edits has a multi-line initializer, grouped and ungrouped alike, since a one-line spec's first line is its whole signature and would pass untrimmed.
One new table row, "a var or const keeps the first line of its signature, a multi-line function keeps all of it", edits each of those members and wants each as a `PackEntry` whose declaration has the real start and end lines (written as a `PackDeclaration` literal, since the `declaration` helper sets `End` equal to `Start`) and: for each `var` and `const`, the first line only (e.g. `var single = map[string]int{`), with no trailing space and no newline; for `Long` and `T.LongMethod`, quarry's whole signature with its newlines.
The row fails if the trim is dropped for any of the four var/const forms (each want has no newline) or applied to every kind (the `Long` and `T.LongMethod` wants keep their newlines).
Read the exact signatures quarry gives with `lyx quarry resolve` on the fixture before writing the wants.

`internal/websterengine/doc.go`'s `# the kick-start pack` section gains one sentence: a `var` or `const` entry shows the first line of its signature only, while its span still covers the whole declaration.

**ImpactSummary:** One branch inside `appendPackAnswer`, whose only caller is `KickstartPack`; it narrows the signature text the fork prompt renders for `var` and `const` entries and changes nothing else.

**Commit:** 1: pack shows var and const signatures by their first line
