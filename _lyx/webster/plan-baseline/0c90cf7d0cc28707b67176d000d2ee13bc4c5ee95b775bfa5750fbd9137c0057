# Card 2 — scope-guard-file-package-cover

**Create:**
- `plan:internal/planglyph#symbolFilesCovered` -> `func symbolFilesCovered(cards []planparser.Card, files []string) bool`
- `plan:internal/planglyph#TestScopeGuard_SymbolCoverage` -> `func TestScopeGuard_SymbolCoverage(t *testing.T)`

**Edit:**
- `internal/planglyph#ScopeGuard`
- `internal/planglyph#targetGlyphUnion`
- `internal/planglyph/doc.go`

**Delete:**
- `internal/planglyph#TestScopeGuard_SymbolInsideTargetsNoFinding`
- `internal/planglyph#TestScopeGuard_SymbolOutsideTargetsOneInformationalFinding`
- `internal/planglyph#TestScopeGuard_HandleTargetCoversTheSymbolItStandsFor`

**Uses:**
- `plan:internal/planparser#RefMember`
- `internal/planparser#RefScope`
- `internal/planglyph#TestScopeGuard_FileLevelCoverage`

**Intent:**
Widen `ScopeGuard`'s symbol check per decision D1: a touched symbol is in scope when it is in the target union as today, when a rename has either side in the union as today, or when a card's file or package target covers every file the symbol lives in.

`symbolFilesCovered` reports whether every path in `files` is covered by some non-member target of `cards`.
For each target it skips a ref `planparser.RefMember(nil, t)` reports as a member, so a member glyph or member `plan:` handle never covers its package's other members, then maps the ref through `planparser.RefScope(nil, t)`.
A file scope covers a path equal to it; a directory scope, a package self glyph or a directory path, covers a path whose `path.Dir` equals it, never one in a subdirectory.
The member test goes through planparser per `PATTERN-ref-shape-registry`; `targetsCover` stays as it is, since the symbol-less file check keeps counting a member target's package directory.

The files a symbol lives in are fixed per delta shape:
- a Created or Deleted symbol: its own `File`;
- a Modified symbol: every `File` on its `Before` and `After` sides, so a symbol moved from a file target into an untargeted file stays reported;
- a Renamed pair: `From.File` and `To.File`, consulted only when neither side is in the union.

The `report` closure takes the symbol's files beside its ID and finding file, and returns early when `symbolFilesCovered(cards, files)` holds.
The finding's `File` and `Detail` stay as today: a Modified symbol's finding file is still `After[0]`, else `Before[0]`, and a Renamed pair's is `To.File`.
The `Detail`'s "own target glyphs" stays true, since a file or package target is spelled as a glyph or path the card declares.
The symbol-less file check and the informational severity are unchanged.

`ScopeGuard`'s doc comment states the new coverage, one sentence per line: a file target covers every symbol in that file, a package target every symbol in a file directly in its directory, a member target only that member, and a symbol with several files is covered only when every one is.
The comment over the Renamed loop, which says a pair is in scope when either endpoint is a declared target, adds that a pair whose `From.File` and `To.File` a file or package target both cover is in scope too.
`targetGlyphUnion`'s doc comment, which calls the union the comparison set every touched symbol is checked against, says instead that it is the exact-match half of the check, with file and package coverage the other.
The `scope-outside-plan` bullet in `internal/planglyph/doc.go` says a card's file or package target covers the symbols in its extent, replacing its "declared targets" wording for symbols.

`TestScopeGuard_SymbolCoverage` is one table test replacing the three single-case symbol-scope tests this card deletes, each of which becomes one of its rows.
Each row is a single card's targets and a delta with no `Files`; the test asserts the reported findings' `File` list exactly, as `TestScopeGuard_FileLevelCoverage` does, and that every finding has check `scope-outside-plan` and `SeverityInformational`.
Rows carried over from the deleted tests:
- member glyph `sub#Foo`, Created `sub#Foo` in `sub/a.go`: none;
- member glyph `sub#Foo`, Created `sub#Bar` in `sub/b.go`: `sub/b.go`, the row that also pins that a member target never covers its package's other members;
- member handle `plan:sub#Foo`, Created `sub#Foo` in `sub/a.go`: none.

New rows:
- file self glyph target `sub/a.go#`, Created `sub#Foo` in `sub/a.go`: none;
- plain file path target `sub/a.go`, Created `sub#Foo` in `sub/a.go`: none;
- file target `sub/a.go#`, Created `sub#Bar` in `sub/b.go`: `sub/b.go`;
- package self glyph `sub#`, Modified `sub#Foo` with `Before` and `After` in `sub/a.go`: none;
- directory path `sub`, Deleted `sub#Gone` in `sub/c.go`: none;
- package self glyph `sub#`, Created `sub/inner#Foo` in `sub/inner/a.go`: `sub/inner/a.go`;
- directory path `sub`, Created `sub/inner#Foo` in `sub/inner/a.go`: `sub/inner/a.go`;
- file target `sub/a.go#`, Modified `sub#Foo` with `Before` in `sub/a.go` and `After` in `sub/b.go`: `sub/b.go`;
- file target `sub/a.go#`, Modified `sub#Foo` with `Before` in `sub/b.go` and `After` in `sub/a.go`: `sub/a.go`;
- file targets `sub/a.go#` and `sub/b.go#`, Modified `sub#Foo` with `Before` in `sub/a.go` and `After` in `sub/b.go`: none;
- file target `sub/a.go#`, Renamed `sub#Foo` in `sub/a.go` to `sub#Bar` in `sub/b.go`: `sub/b.go`;
- file target `sub/a.go#`, Renamed `sub#Foo` in `sub/b.go` to `sub#Bar` in `sub/a.go`: `sub/a.go`;
- package self glyph `sub#`, Renamed `sub#Foo` in `sub/a.go` to `sub#Bar` in `sub/b.go`: none;
- file-unit member glyph `sub/a.go#Foo`, Created `sub#Bar` in `sub/a.go`: `sub/a.go`;
- member handle `plan:sub#Foo`, Created `sub#Bar` in `sub/a.go`: `sub/a.go`.

The remaining `ScopeGuard` tests keep their verdicts unchanged, covering the rename either-side rule, a Deleted and a Modified symbol outside the union, the empty delta and the symbol-less file check.
The card adds one top-level test and deletes three, so the `internal/planglyph` row of `cmd/lyx/testdata/test-budget.yaml` needs no change.

**ImpactSummary:** `ScopeGuard` keeps its signature and stays informational; its callers, `batchDelta.ScopeGuard` and the websterengine test fake, now get fewer findings for symbols in file or package targets, so record-batch's digest `deviations` shrink accordingly, and the three deleted tests have no callers.

**Commit:** 2: ScopeGuard counts file and package targets as covering their symbols
