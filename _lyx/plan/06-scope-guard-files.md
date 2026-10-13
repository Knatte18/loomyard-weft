# Card 6 — The scope guard names each finding's file and reports changed files no target covers

**Create:**
- `internal/planparser#RefScope`
- `internal/planglyph#TestScopeGuard_FileLevelCoverage`

**Edit:**
- `internal/planindex#Finding`
- `internal/planglyph#ScopeGuard`
- `internal/planglyph#TestScopeGuard_SymbolOutsideTargetsOneInformationalFinding`
- `internal/planglyph/doc.go`
- `internal/planparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D4's guard half: the guard's finding carries its file as a structured field, and the guard also reports, at file level, a changed file the delta gives no symbols for unless a card target covers it.
`ScopeGuard` stays informational (`SeverityInformational`, check `scope-outside-plan`) and its signature is unchanged.

`planindex.Finding` gains `File string`, the worktree-relative file a finding concerns, empty for a finding about no file; `Error()` is unchanged, and only `ScopeGuard` sets it.

`RefScope` lives in `internal/planparser/targetdirs.go` beside `RefFile`, so every glyph-to-path decision stays in `planparser` (`PATTERN-glyph-conversion-chokepoint`).
It maps one target ref through the same `refUnit` `RefFile` and `CardTargetDirs` use: a unit or path with a file extension returns that cleaned file with `isFile` true, compared by equality; any other unit returns its package directory with `isFile` false, a glyph's or `plan:` handle's unit taken through `goPackageDir`, the one external-test unit mapping, so quarry's `<dir>_test` pseudo-unit names `<dir>`.
`ok` is false for a ref that names no unit; a nil plan reads as the default language, as `RefFile`'s does.

`ScopeGuard`:
- Every symbol finding it makes today also sets `File` to the file it already names in `Detail`.
- It collects the files the delta gives symbols for (`Created`, `Deleted`, `Modified` before and after, and both sides of `Renamed`).
- For each entry of `delta.Files` whose path is not among them, it reports one finding with `File` set to the path and a detail naming the file as changed outside the completed batch's own targets with no symbol in the delta, unless some card target's `RefScope` covers it: a file scope equal to the path, or a package directory equal to `path.Dir` of the path, so a file in a subdirectory of the package is not covered.
  Every disposition counts, unsupported and extraction-error included, and non-Go files directly in a covered package directory are covered.
- Its doc comment states that rule and the file-granular bound: it lets through a symbol-less change in a target's file or package directory and reports every other one.

`TestScopeGuard_SymbolOutsideTargetsOneInformationalFinding` also asserts `got[0].File == "sub/b.go"`.

`TestScopeGuard_FileLevelCoverage` is a table over `delta.Files` entries with no symbols in the delta, each row naming the card targets and the wanted `File` list (empty for none):
- an import-only change in `sub/a.go` under symbol target `sub#Foo`: none;
- the same in `sub/inner/a.go`: `sub/inner/a.go`;
- `sub/a_test.go` under `sub_test#TestFoo`, an external test package's symbol target: none;
- `docs/notes.md` under file target `docs/notes.md`: none, and `docs/other.md` under it: `docs/other.md`;
- `sub/README.md` under package target `sub#`: none;
- `other/x.go` with disposition error under `sub#Foo`: `other/x.go`;
- a `plan:sub#Bar` handle target covering `sub/new.go`: none;
- a file that does carry an out-of-target symbol yields only that symbol's finding, never a second file-level one.
It lands in `internal/planglyph/scope_test.go`, `package planglyph`, which adds one top-level test, so `cmd/lyx/testdata/test-budget.yaml`'s `internal/planglyph` row rises by one.

`internal/planglyph/doc.go`'s `scope-outside-plan` entry adds the file-level finding; `internal/planparser/doc.go`'s sentence naming `goPackageDir` beside `Glyph.UnitPath` names `RefScope` among its callers.

**ImpactSummary:** One struct field nothing but the guard sets, one exported planparser mapping beside `RefFile`, and the guard's own file-level pass; record-batch still receives findings through the unchanged `planindex.Delta.ScopeGuard` until card 7 routes them.

**Commit:** 6: Report scope findings by file, including changed files no target covers
