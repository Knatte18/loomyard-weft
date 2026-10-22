# Card 24 — planparser TargetHolds

**Create:**
- `plan:internal/planparser#TargetHolds` -> `func TargetHolds(plan *Plan, target, member, file string) bool`
- `plan:internal/planparser#TestTargetHolds` -> `func TestTargetHolds(t *testing.T)`

**Uses:**
- `internal/planparser#RefMember`
- `internal/planparser#goPackageDir`

**Intent:**
D9 step 3's overlap rule, in the sole reader of plan refs (PATTERN-sole-parsers, PATTERN-ref-shape-registry, PATTERN-glyph-conversion-chokepoint), so the trace never parses a ref itself.

`TargetHolds(plan, target, member, file)` reports whether a card target holds a code location: `member` is the glyph quarry found there, empty for a location inside no member, and `file` is the location's repository-relative path.
It is true when:
the target maps through `RefMember` (a member glyph, or a `plan:` handle as the glyph it becomes) to `member` itself, or to a type `member` is a member of (an owner-chain prefix, as `pkg#T` holds `pkg#T.M`);
the target is the self glyph of `member`'s unit, with an external test unit `<dir>_test` read as `<dir>` through `goPackageDir`, the existing adjustment;
or the target is `file` as a path or as its file self glyph.
A `plan:` handle that `RefMember` does not map, one whose body is a self glyph, compares as its body, so a handle naming a file a card creates holds a location in that file.
A target that maps to nothing above, such as a file self glyph of another file, is false, and a nil plan reads as the default language.
Matching a member target that only resolves into `file` is the trace's job (card 25), since it needs quarry.

`TestTargetHolds`, table-driven in package `planparser`, rows: the member glyph itself; a `plan:` handle whose body is the member; a containing type; a method's sibling (false); the unit self glyph; an external test member against its directory's unit self glyph; the file as a path and as a file self glyph with an empty member; a file target against a found member in that file; a `plan:` handle whose body is that file's self glyph; another file (false); a `language: none` plan (only the path rule holds).
