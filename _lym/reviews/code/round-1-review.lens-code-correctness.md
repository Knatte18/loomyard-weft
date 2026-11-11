```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `internal/testcover` selector diff, run case by case from the context I already hold.

**Per function or branch: case run, correct or not**
- `Select`, `mapMiss`: missing, unreadable, build-failure, unmarked and marker-mismatch maps, and a test index outside the `Funcs` table. All give the right miss. Correct.
- `Select` flow, `ownFilesMiss`, `commitMiss`: an own test file edited gives `MissOwnFiles`. A commit rev the object store cannot read gives `MissCommitUnreadable`. Both correct. An empty `m.Files` skips the probe, but the first linked-file read then returns `MissCommitUnreadable`. Correct.
- `linkedChanges`, `changedFiles`, `ownedFiles`: a linked file added, removed, rehashed, untracked or nested all come out right. A linked test file is skipped. Method moves between files are matched across the package. Correct.
- `compareGoFile`: a commit read error returns a miss and is not cached. A working-file read error propagates as an error. The cache key `commit\x00rel` is safe because maps built from the same commit record the same hashes. Correct.
- `compareGoSources`: I ran each of these.

  | Case | Result |
  |---|---|
  | Comment or formatting edit | no change, correct |
  | Reordered or regrouped imports | no change, correct |
  | Directive change | miss, correct |
  | Import removed | miss, correct |
  | Const group changed or spec added | miss, correct |
  | New iota group | miss, correct |
  | Const, type or var without initializer added | no miss, correct |
  | Var with initializer added | miss, correct |
  | `init` count up | miss, correct |
  | `max` or a predeclared type added | miss, correct |
  | Parse failure on either side | miss, correct |
  | Two `init` functions changing together | grouped, correct |
  | File deleted | every function changed, correct (but see F3) |

- `methodSetMiss`, `signaturesEqual`: value to pointer receiver, method added or removed, and an unnamed or multi-name parameter list all behave correctly.
- `equalSyntax`: comments and positions are ignored while `type A = B` is told from `type A B`. No panic path found. Correct.
- `otherFileMiss`, `embedPatterns`, `parseEmbedPatterns`, `embedded`: a directory pattern, a glob, a quoted pattern, an `all:` prefix and a malformed line all behave correctly. A directory pattern covers files below it. Two gaps follow in F1 and F2.
- `selectTests`: nothing changed gives nil. A spawn-marked test is selected whenever any function changed. Names are sorted and compacted. Correct.
- `NestedPackageDirs` move: code is identical to the old `nestedPackageDirs`. Correct.

**Findings**

- **F1, MEDIUM, design:** `embedPatterns` reads only non-test Go files (`isGoSourceFile`, `internal/testcover/testselect.go:424`).
  - For the importing package's own files, `ownFilesMiss` (`internal/testcover/testselect.go:200`) applies the same carve-out.
  - An own `_test.go` that `go:embed`s a file `inert_paths` matches, such as `testdata/x.md`, is therefore treated as inert.
  - An edit to that file gives no miss and narrows the run, which D5 forbids.
  - Fix: for the own package, also read the patterns from the working tree's `_test.go` files.

- **F2, LOW, design:** `importAllowed` treats an import path whose first segment has no dot as standard library (`internal/testcover/testselect.go:393`).
  - A non-stdlib dotless module path other than `ModulePath`, such as a `replace`d module or workspace sibling, is wrongly allowed instead of missing.
  - Fix: test membership in the standard library explicitly, for example `go/build`'s `Context.Import` with `ImportComment`, or `golang.org/x/tools/go/packages`' stdlib list, or `internal/stdlib` data.

- **F3, LOW, design:** deleting a linked Go file that imports anything gives `MissNonBody` ("removes the import"), through `compareGoSources` at `internal/testcover/declcompare.go:80-84`.
  - The same happens for a deleted file with a directive, const, var or type.
  - D4 says a deleted file makes all its functions changed, and the acceptance criterion says it selects the tests that covered a block of it.
  - `DeletedFile` passes only because its fixture `a/gone.go` has no imports.
  - Most real Go files import something, so a real deletion nearly always runs the package whole.
  - This is safe because it widens, but it conflicts with the D4 criterion.
  - Fix: when `after.present` is false, skip the import, value, directive and predeclared checks.
  - Alternatively, record the reading in the decision record.
  - Add a fixture row for a deleted file that imports a package.

**Tool calls:** none made; the context already held every file read.
