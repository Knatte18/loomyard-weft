```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran one concrete case through each function and branch the diff adds or changes.

**Per function or branch, the case run, and whether it is correct**

- **`baseOverlay`, test-file skip** (`internal/gatecli/redfirst.go:367`): `sub/a/a_test.go` is skipped before the non-Go branch. Correct.
- **`baseOverlay`, file outside the module or under `testdata`**: `other/x.go` and `sub/a/testdata/data.json` give `isModulePackage` false, so both are skipped. Correct for the module bound; see finding 1 for embedded `testdata`.
- **`baseOverlay`, non-Go file embedded by name**: `sub/a/data.json` with base `a.go` holding `//go:embed data.json`. `embeddedAtBase` returns true, `base.file` reads the file, the copy is written and mapped. Correct.
- **`baseOverlay`, embedded file the base lacks**: `sub/a/embedded/new.json` under `all:embedded`. It counts as embedded, `found` is false, and it maps to `""`. The go command's overlay deletion reaches the embed directory walk, so the base build does not see the file. Correct.
- **`baseOverlay`, file deleted at HEAD that the base embeds**: it is in `changed`, the base holds it, and it maps to a copy that the overlay adds back. Correct.
- **`baseOverlay`, error paths**: an `embeddedAtBase` error and a `base.file` error both propagate. `redFirstCall.run` prints them with "re-run the same command" and exits 1. Nothing is left half-made, because the deferred `RemoveAll` clears the temp directory. Correct.
- **`embeddedAtBase`, walk up to the module root**: `sub/a/embedded/deep/data.json` with moduleRel `sub`.
  - At `deep` and `embedded` there are no Go files.
  - At `sub/a`, `below` is `embedded/deep/data.json`, which matches `embedded`. The walk stops at `sub`.
  - Correct.
- **`embeddedAtBase`, moduleRel `.` and a file at the root**: `README.md` gives dir `.`, and `TrimPrefix(rel, "./")` leaves the path unchanged as `below`. The walk ends at `.`. Correct.
- **`embeddedAtBase`, base Go file listed but unreadable or absent**: a `base.file` error propagates. With `found` false, the source is nil and yields no patterns. Correct.
- **`embeddedAtBase`, test file that embeds**: `fixture.json` is embedded only by `a_test.go`, which is skipped, so the result is false. Correct.
- **`embedPatterns`**:
  - The line `//go:embed data.json all:embedded "q*.txt"` gives `[data.json embedded q*.txt]`. Correct.
  - An indented directive is accepted. Correct.
  - A directive line inside a raw string or block comment over-matches. That only restores a file the build does not read, which is harmless.
  - A quoted pattern with a space, `"a b.txt"`, splits into `a` and `b.txt` (see finding 2).
- **`embedPatternMatches`**:
  - `inner/*.txt` against `inner/embedded.txt` matches.
  - `embedded` against `embedded/deep/data.json` matches through its parent candidates.
  - `*` against a top-level file or directory matches.
  - A malformed pattern returns false from `path.Match` and matches nothing.
  - An empty `below` is unreachable: `rel` is a file path, so `below` is never empty.
  - Correct.
- **`redFirstCall.run`**: the unrestorable branch is gone and the overlay error path is unchanged.
  - Red: some test listed in `red` gives exit 0.
  - Green: every tier exits 0, so `unjudgedCode` stays 0 and the exit is `RedFirstGreenExit`.
  - Unjudged: a nonzero tier exit with no red test gives exit 1.
  - Exit 5 now comes only from `runRedFirst`'s scoper check (`redfirst.go:127-129`).
  - Correct.
- **`redFirstFailure`, exit 5 branch** (`internal/loomshed/redfirstgate.go:101-103`): the text names only the scoper and the way forward is unchanged. Correct, since 5 now has the one cause.

**Findings**

1. **LOW, design**, `internal/gatecli/redfirst.go:367` (`baseOverlay`'s `isModulePackage` filter on a non-Go file).
   A changed non-Go file that a production Go file embeds keeps its working-tree text when it lies under a `testdata`, `_`-prefixed or `.`-prefixed directory.
   Go accepts an explicit pattern such as `//go:embed testdata/golden.txt` or `//go:embed _static/index.html`, and `all:dir` also reaches `_`/`.` subdirectories.
   `isModulePackage(module.rel, path.Dir(rel))` is false for those directories (`internal/gatecli/redundancytests.go:191-193`), so `embeddedAtBase` is never asked.
   On the base, the test then reads the fixed text, passes, and the call exits `RedFirstGreenExit`. That pushes the run back to a "No test seam" waiver, the same failure the task removes.
   The new `internal/gatecli/doc.go` line "An embedded file lies outside `testdata`" states as fact a restriction Go does not impose.
   Fix: for non-Go files, test only the module bound and let `embeddedAtBase` decide. Then `testdata` stays left alone because nothing embeds it, not by path. Alternatively, drop the doc's claim.
2. **NIT, design**, `internal/gatecli/redfirst.go:437-438` (`embedPatterns`).
   `strings.Fields` splits a quoted pattern containing a space, such as `//go:embed "my file.txt"`, into `"my` and `file.txt"`. A changed file with a space in its name is therefore never found embedded and keeps its working-tree text.
   Go's directive grammar allows quoted patterns precisely for such names.
   Fix: parse the quoted and back-quoted fields as Go string literals (`strconv.Unquote`) before splitting on whitespace.

Tool calls:
1. `grep` of `CodeDirAt` and `CodeFileAt` in `internal/fabricengine/codereads.go`: my context lacked how the base tree reports a missing file or directory and the `.` root, which `embeddedAtBase`'s error and absent paths depend on.
