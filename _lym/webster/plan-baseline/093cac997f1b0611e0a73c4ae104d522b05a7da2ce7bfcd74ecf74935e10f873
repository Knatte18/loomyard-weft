# Card 1 — scankit-banned-imports

**Create:**
- `internal/testkit/scankit#AssertNoImports`
- `internal/testkit/scankit#bannedImports`
- `internal/testkit/scankit#TestBannedImports`
- `internal/testkit/scankit#recordingTB`

**Uses:**
- `internal/testkit/scankit#writeTree`
- `internal/testkit/scankit#RequireFloor`
- `internal/testkit/scankit#Root`
- `internal/testkit/scankit#importViolations`

**Intent:** Add the one import check `scankit` lacks: a ban on named import paths, the standard library included.
`AssertImportAllowlist` admits every standard-library import, so it cannot fail a leaf package that imports `os/exec` or `os`;
the leaf tests of `internal/fabriclayout` and `internal/fabriccore` (cards 2 and 12) need that failure (decision record D1 and its first acceptance criterion).

`bannedImports` parses each non-test `.go` file directly in `dir` with `parser.ImportsOnly`, as `importViolations` does, skipping `_test.go` files and subdirectories.
A `banned` entry is an exact import path, or a path ending in `/...` that bans every import under that path, so `os/...` bans `os/exec`, `os/signal` and `os/user` but not `os` itself.
It returns one `<file>: <path>` violation per banned import, sorted in directory order, and the count of files it parsed.
A missing directory and an unparsable file return the error.
`AssertNoImports` resolves `pkgDir` against `Root` the way `assertImports` does, fails the test with `t.Fatalf` on the error, calls `RequireFloor` on the parsed count so a scan of an empty directory fails loudly, and reports each violation with `t.Errorf`, naming the package and the banned path.

`TestBannedImports` is tier0 and table-driven over trees `writeTree` builds, each row's expected violations or messages a literal:
- `bannedImports` rows: a file importing `os/exec` and one importing `os` are each reported when banned exactly;
  `os/...` reports `os/exec` and not `os`;
  a banned non-stdlib path is reported;
  an import outside the banned list, a `_test.go` file and a file in a subdirectory are not;
  a clean tree reports nothing with its parsed count;
  a missing directory and an unparsable file return an error.
- `AssertNoImports` rows run it on a `recordingTB`, a test-file type embedding `testing.TB` that overrides `Helper` as a no-op and `Error`, `Errorf` and `Fatalf`, recording each message and the failed state, so no call reaches the nil embedded `testing.TB`.
  `AssertNoImports` returns right after its `t.Fatalf` on a scan error, since a recorded `Fatalf` does not stop the call.
  A missing directory records exactly one fatal message and no floor message, an empty directory records the floor failure, and a banned import records one error naming the package and the path.

**Commit:** `1: scankit-banned-imports`
