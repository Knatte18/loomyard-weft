# Card 1 — testreach-changed-funcs

**Create:**
- `internal/testreach/doc.go`
- `internal/testreach/testmain_test.go`
- `plan:internal/testreach#Module` -> `type Module struct`
- `plan:internal/testreach#FileSet` -> `type FileSet map[string][]byte`
- `plan:internal/testreach#ChangedFunc` -> `type ChangedFunc struct`
- `plan:internal/testreach#ChangedFuncs` -> `func ChangedFuncs(module Module, base, head FileSet) ([]ChangedFunc, error)`
- `plan:internal/testreach#TestChangedFuncs` -> `func TestChangedFuncs(t *testing.T)`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/testcover#FuncKey`
- `internal/testcover#ContentHash`

**Intent:**
Open the new package `internal/testreach`, the one cross-package reader of the coverage maps (D1, D2).
This card lands its D2 half: the finder of a diff's changed functions.
The package runs no git: every caller hands in the file contents of both sides.

`Module` names the Go module the maps hold: `Dir`, the absolute module root; `Rel`, its worktree-relative slash path; `Path`, its import path.
`FileSet` maps a worktree-relative slash path to a file's contents.
A caller puts in each set every Go file of each package whose files it wants compared, so a function that moved between files of one package is seen on both sides.

`ChangedFuncs(module, base, head)` compares the two sets function by function and returns the changed functions sorted by `testcover.CompareFuncKeys`.
- A `ChangedFunc` carries `Key testcover.FuncKey` (import path, file name, function name with `<Type>.<Method>` for a method, spelled as `testcover`'s block keys spell it),
  `Hash`, the `testcover.ContentHash` of the function's declaration text, and `New`, set when the base holds no function it matches.
- Only production Go files count: a `_test.go` path, a path outside the module, and a path under `testdata` or a directory opening with `.` or `_` are passed over.
- A function is matched across the sets by package and name, whatever its file, so a function moved verbatim to another file of its package is unchanged.
- A name the package defines in more than one file of either side, as build-constrained variants do, is matched by package, file and name instead.
- The `init` functions of one file are matched by package, file and their order in that file.
- A function is changed when it is new or its text differs; a function only the base holds is no changed function.
- A file that does not parse is an error naming the file.

`TestChangedFuncs` is one tier0 table over in-memory `FileSet`s, one row per D2 criterion:
a new function and a body change are found (the new one marked `New`);
a function whose lines moved with unchanged text, a function moved verbatim to another file of its package, a deleted function and a test file's function are not;
a change to one of two build-constrained variants of a function is found in that variant's file alone;
the second `init` of a file changed while the first did not is found alone;
a method's key is `<Type>.<Method>`;
a changed function in a file outside the module, under `testdata`, or under a directory opening with `.` or `_` is not found;
a file that does not parse returns an error.
`internal/testreach/testmain_test.go` runs the package's tests under `tmuxkit.Main`, as every package does; it is `untagged`.

`doc.go` documents the package: the reader to come and this card's finder, with the matching rules above.
`docs/overview.md` gains a line for `internal/testreach/` beside `internal/testcover/`.

**ImpactSummary:** A new package with no callers yet and one new line in the overview's package tree; nothing existing changes.
