# Card 24 — planparser TargetHolds

**Create:**
- `internal/planparser#TargetHolds`
- `internal/planparser#TestTargetHolds`

**Uses:**
- `internal/planparser#RefMember`
- `internal/planparser#goPackageDir`
- `internal/planparser#ParseRef`
- `internal/planparser#ParseMintedCSharpID`
- `internal/planparser#PathEntryPath`
- `internal/planparser#HandleBody`

**Intent:**
D9 step 3's overlap rule, in the sole reader of plan refs (PATTERN-sole-parsers, PATTERN-ref-shape-registry, PATTERN-glyph-conversion-chokepoint), so the trace never parses a ref itself.

`TargetHolds(plan, target, member, file)` reports whether a card target holds a code location: `member` is the glyph quarry found there, empty for a location inside no member, and `file` is the location's repository-relative path.
It is true when:
the target maps through `RefMember` (a member glyph, or a `plan:` handle as the glyph it becomes) to `member` itself, or to a type `member` is a member of, decided on the parsed glyphs' unit, owner chain and name, compared as `tools/plandag`'s `columnsOverlap` compares them (`ParseRef` for a plan ref, `ParseMintedCSharpID` for a C# member id quarry minted), never by a string prefix or split of either glyph string (PATTERN-glyph-conversion-chokepoint);
the check runs one way, unlike `columnsOverlap`'s symmetric overlap: the units are equal and the target's owner chain plus name is a prefix of `member`'s, never the reverse, so `pkg#T` holds `pkg#T.M` and `pkg#T.M` does not hold `pkg#T`;
the target is the self glyph of `member`'s unit, with an external test unit `<dir>_test` read as `<dir>` through `goPackageDir`, the existing adjustment;
or the target is `file` as a path or as its file self glyph.
The file rule reads a target through `PathEntryPath` and compares the path it returns with `file`, compares a target `PathEntryPath` does not read with `file` as a plain path, and reads a `plan:` handle's body through `HandleBody`; it never builds a self glyph from `file` or compares a self glyph by string.
A `plan:` handle that `RefMember` does not map, one whose body is a self glyph, compares as its body, so a handle naming a file a card creates holds a location in that file.
A target that maps to nothing above, such as a file self glyph of another file, is false, and a nil plan reads as the default language.
Matching a member target that only resolves into `file` is the trace's job (card 25), since it needs quarry.

`TestTargetHolds`, table-driven in package `planparser`, rows: the member glyph itself; a `plan:` handle whose body is the member; a containing type; a member target against its owning type as `member` (false); a method's sibling (false); the unit self glyph; an external test member against its directory's unit self glyph; the file as a path and as a file self glyph with an empty member; a file target against a found member in that file; a `plan:` handle whose body is that file's self glyph; another file (false); a `language: none` plan (only the path rule holds).
