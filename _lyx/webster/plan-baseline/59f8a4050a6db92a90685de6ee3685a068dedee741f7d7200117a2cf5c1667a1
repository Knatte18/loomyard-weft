# Card 1 — planparser decides when one ref covers another

**Create:**
- `internal/planparser#RefCovers`
- `internal/planparser_test#TestRefCovers`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#RefScope`
- `internal/planparser#RefFile`
- `internal/planparser#refUnit`
- `internal/planparser_test#TestRefFile`

**Intent:**
Add `RefCovers` to `internal/planparser/targetdirs.go`, the ref-shape decision D1 of the decision record needs, so neither the batcher nor webster tests a ref's shape locally (`PATTERN-ref-shape-registry`, `PATTERN-glyph-conversion-chokepoint`).
Name it in the file's header comment beside `CardTargetDirs` and `RefFile`.

`RefCovers(plan, cover, target)` reports whether `cover`, a Rename pair's old side or a Delete target of a later card, covers `target`, a ref an earlier card writes.
Its doc comment states the three cover shapes below and that a ref mapping to no unit neither covers nor is covered.
A nil plan reads as the default language, as `RefScope` does.

- A cover naming a member, a glyph or a `plan:` handle whose body parses to a glyph that is not a self glyph, covers a target naming the same member: both sides parse through `parseGlyph` (a handle through its `HandleBody`), and they compare by `Glyph.String`.
  So `internal/a#Foo` covers `plan:internal/a#Foo`, and covers neither `internal/a#Bar` nor `internal/a/a.go#`.
- A cover naming a file, a path or self glyph whose unit has a file extension, covers a target whose `RefScope` is that same file (a plain path, a file self glyph or a member glyph or handle in that file unit), and a target that came from a glyph or handle (member or self) whose `RefScope` is a package directory equal to the file's `path.Dir`.
  Take "came from a glyph" from `refUnit`'s `fromGlyph`, so a plain directory path such as `internal/a` is not covered by a file inside it.
  An external test package unit maps through `RefScope`'s `goPackageDir`, so `internal/a/a_test.go#` covers `internal/a_test#TestRun`.
- A cover naming a directory, a plain directory path or a package self glyph (a `RefScope` with isFile false that is not a member), covers a target whose directory lies at or under it: the target's `RefScope` itself when isFile is false, its `path.Dir` when it is a file.
  "Under" is a path-segment prefix, so `internal/a` covers `internal/a/sub/s.go#` but not `internal/ab/x.go#`; a cover of `.` covers every mapped target.

Write a small unexported helper beside it for the member test if that reads better, kept in the same file.

`TestRefCovers` is a new table test in `internal/planparser/targetdirs_test.go` over `&planparser.Plan{Language: "go"}`, one row per pair, asserting the exact bool:
member covers the same member through a handle (true); member against another member of the package (false); member against the file self glyph of its file (false);
file self glyph against the plain path of the same file (true) and a member glyph in that file unit (true);
file self glyph against a package-unit member glyph, a package-unit handle and the package self glyph of its directory (each true);
file against another file of the same directory (false); file against the plain directory path of its directory (false);
`internal/a/a_test.go#` against `internal/a_test#TestRun` (true);
directory path against a file glyph under a subdirectory (true), a package-unit member glyph of that directory (true) and a file under a sibling sharing the prefix `internal/ab` (false);
package self glyph `internal/a#` against a member glyph in its file `internal/a/a.go#Run` (true);
an unmappable handle `plan:nohash` as cover against `internal/a#Foo` (false) and as target under the cover `internal/a` (false).

The package has no free row in `cmd/lyx/testdata/test-budget.yaml`: raise `internal/planparser` by one.

**ImpactSummary:** Adds one exported pure function and its test; no existing behavior changes, and the budget row rises by one.

**Commit:** 1: planparser decides when one ref covers another
