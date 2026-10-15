# Card 1 — planparser-ref-member

**Rename:**
- `internal/planparser#refMember` -> `plan:internal/planparser#RefMember`

**Create:**
- `plan:internal/planparser_test#TestRefMember` -> `func TestRefMember(t *testing.T)`

**Edit:**
- `plan:internal/planparser#RefMember`
- `internal/planparser#RefCovers`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser_test#TestRefFile`

**Intent:**
Export `refMember` as `RefMember(plan *Plan, ref string) (string, bool)`, so `internal/planglyph`'s scope guard can tell a member ref from a self glyph, file path or directory path through planparser's own classification, per `PATTERN-ref-shape-registry`.
Its body stays as it is, with one addition: a nil plan reads as the default language, the same opening `RefFile`, `RefScope` and `RefCovers` carry, since today `planLanguage` dereferences the plan and `ScopeGuard` holds no plan.
Its doc comment keeps its two sentences and adds the nil-plan sentence, one sentence per line.
`RefCovers` calls `RefMember` at both of its call sites in place of `refMember`; its behavior does not change, and `TestRefCovers` keeps passing unchanged.
The file header comment of `targetdirs.go`, which names the file's exported mappings, adds `RefMember` to that list.

`TestRefMember` is a table-driven test in the external `planparser_test` package beside `TestRefFile`, in its shape, with `t.Parallel()` on the test and each row.
Its rows, each asserting both the returned string and `ok` exactly:
- a package-unit member glyph `internal/a#Foo` returns `internal/a#Foo`, true;
- a file-unit member glyph `internal/a/a.go#Run` returns `internal/a/a.go#Run`, true;
- a member `plan:` handle `plan:internal/a#Foo` returns `internal/a#Foo`, true;
- a package self glyph `internal/a#` returns false;
- a file self glyph `internal/a/a.go#` returns false;
- a plain directory path `internal/a` returns false;
- an unmappable handle `plan:nohash` returns false;
- a non-Go plan (`Language: "none"`) with `internal/a#Foo` returns false;
- a nil plan with `internal/a#Foo` returns `internal/a#Foo`, true, the row that reaches the new nil-plan default.

The `internal/planparser` row of `cmd/lyx/testdata/test-budget.yaml` rises by one for `TestRefMember`.

**ImpactSummary:** One unexported helper becomes exported; its sole callers are the two calls in `RefCovers`, same package, so no behavior outside planparser changes.

**Commit:** 1: planparser exports RefMember
