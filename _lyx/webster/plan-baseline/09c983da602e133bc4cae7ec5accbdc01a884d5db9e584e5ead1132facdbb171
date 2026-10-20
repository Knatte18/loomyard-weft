# Card 4 — module-guard-scan

**Create:**
- `internal/impactset#GuardScanKind`
- `internal/impactset#GuardMisplaced`
- `internal/impactset#GuardNotIntegration`
- `internal/impactset#GuardParse`
- `internal/impactset#ModuleGuardTests`
- `internal/impactset#TestModuleGuardTests_OverAModule`

**Edit:**
- `internal/impactset#GuardScanError`
- `internal/impactset#guardNamesInFile`
- `internal/impactset/doc.go`
- `internal/impactset#TestFindGuardTests`

**Uses:**
- `internal/impactset#FindGuardTests`
- `internal/impactset#writeTestFile`
- `internal/impactset#writeRepoFile`

**Intent:** Give `lyx gate test --guards` (Card 6) the module-wide guard scan D5 names, and a guard-scan error it can turn into the way forward of its kind.

`GuardScanError` gains two fields: `Kind GuardScanKind`, one of `GuardMisplaced` (a marker not directly above a top-level `func Test…` line), `GuardNotIntegration` (a marker in a file no `integration`-tier run compiles) and `GuardParse` (a file `parser.ParseFile` rejects, a file that cannot be read included, since `ParseFile` returns the read failure as is);
and `Line int`, the marker's line for the two marker kinds and the parser's line for `GuardParse` when the parser reports a position, 0 otherwise.
`guardNamesInFile` fills both; the `Error` text is unchanged, so the round gate's findings and webster's card gate read as before.

`ModuleGuardTests(moduleDir)` runs `go list -e` under the `integration`, `tmux` and `llm` tags over `./...` at `moduleDir` as a logged spawn, as `listPackages` does, reading each entry's directory and error.
`-e` keeps a package whose test file cannot be read in the list, so the scan reaches that file and returns a `GuardParse` error naming it with the operating system's error and no line.
An entry with no directory, which is how `go list -e` reports an unreadable package directory (`pattern ./...: open c: permission denied`), and a failed `go list` run are returned as a plain error naming `moduleDir` and the enumeration's error, which names the directory.
Otherwise it returns `FindGuardTests` over the listed directories made module-relative, keyed by those directories.
`Derive` keeps its own `listPackages` and its fallback on a failed `go list`.

`TestFindGuardTests` gains two row fields, `wantKind` and `wantLine`, asserted on every error row's `GuardScanError`: each misplaced-marker row `GuardMisplaced` with its marker's line (3, and 4 for the floating-body row), the integration-tier row `GuardNotIntegration` with the marker's line, and the parse row `GuardParse` with the non-zero line the parser reports for its truncated function.
`TestModuleGuardTests_OverAModule`, in an `integration`-tagged file, writes a module with a package `g` holding one marked test and a package `p` holding none, and asserts, as steps over that module: the scan returns `{"g": ["TestGuard"]}`; with `p/p_test.go` made unreadable by `chmod 000` the error is a `GuardParse` `GuardScanError` naming `p/p_test.go` with line 0; with that restored and the directory `c` made unreadable, the error is no `GuardScanError` and names `moduleDir` and `c`; a directory with no `go.mod` returns an error naming that directory.
The unreadable steps skip when the test runs as root, which reads any file.

**ImpactSummary:** impactset's guard-scan error gains two fields and the package gains one exported scan; the error text, Derive and both existing callers of FindGuardTests are unchanged.
