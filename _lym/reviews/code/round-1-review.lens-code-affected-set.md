```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found and whether the diff covers it.**

1. **`showFile` (`internal/fabricengine/configchanges.go:116`)**
   - Callers found: `configchanges.go:104`, `upstreamrun.go:72`, `recordsreads.go:193` (via `recordsFilesAt`), and `anchormarker.go:94` (via `committedMarker`).
   - The revs those callers pass are HEAD, tips, SHAs and a tracking ref, which `ResolveSHA` or the direct SHA path handles. Covered.
   - No caller type-asserts `*gitexec.GitError` on a `showFile` error. The only `GitError` use is `recordsreads.go:180`, which is a config read, not `showFile`. Covered.
   - The directory-path answer changes from the tree listing to nil. No caller asks for a directory, and the summary records it. Covered.
   - Tests and fixtures: `ShowFileForTest` in `export_test.go` and `TestShowFile_ReadsCommittedBytes` cover it. Existing `ReadConfigChanges` and anchor tests exercise the callers. Covered.
   - Docs: the `fabricengine/doc.go` addition is accurate. No `ls-tree` or `git show` text is left stale in either `doc.go`. Covered.

2. **`gitkit.RevParse(HEAD)` (`internal/gitkit/query.go`)**
   - Callers found: `CommitFile` and many test call sites. Every caller of HEAD gets the same SHA, and git still answers every non-HEAD rev. Covered.
   - The `internal/gitkit/doc.go` leaf list and the `leaf_enforcement_test.go` allowlist both gained `dotgit`. Covered.

3. **PATTERN change**
   - `pattern/PATTERN-leaf-packages.md` now lists `dotgit` for `gitkit`, matching `doc.go` and the allowlist test.
   - The `dotgit` entry in the same file says it imports only the standard library, so `gitkit` reaching `dotgit` adds no feature package. Covered.
   - The top-level `PATTERN.md` entry for `PATTERN-leaf-packages` is a one-line rule that names no import list, so no change is needed there. Covered.

4. **Commit messages and departures:** No `Pattern-Departure:` trailer is needed. Nothing in the diff departs from the decision record's scope, which limits changes to `fabricengine`, `gitkit` and `pattern/`.

**Findings:** none.

**Tool calls**
1. Grep and read of the `showFile` callers and `configchanges.go:85-110`, plus a search for other import-list docs. My context lacked the full caller set.
2. Grep in `PATTERN.md` and `pattern/PATTERN-lym-files.md` for `lymtemp`, plus reads of `anchormarker.go:80-100` and `recordsreads.go:180-196`. My context lacked how those callers use `rev`.
3. Grep for `committedMarker` and `recordsFilesAt` callers and for `GitError` use. My context lacked the callers' rev values and any error-type checks.
4. Grep for `ls-tree`, `git show` and `rev-parse HEAD` in the two `doc.go` files and `docs/*.md`. My context lacked a stale-text check.
