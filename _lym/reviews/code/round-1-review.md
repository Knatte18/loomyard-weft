---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shuttlecli/cli.go:87
    summary: no test pins that await-forks alone resolves through ResolveWithin, or that it runs from a weft cwd
    fix: In internal/shuttlecli/cli_test.go, beside TestAwaitForksVerb, add a row that runs await-forks with a cwd in the weft sibling's _lym/plan and expects it to pass the resolution gate. Add a second row where another strict verb such as state, run from the same cwd, is refused with the outside-anchor error.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/lymcwd/lymcwd_test.go:336
    summary: TestResolveWithin has no row for a worktree named with the weft suffix and no owner beside it, or for a checkout with no recorded marker
    fix: In TestResolveWithin, add a row for a checkout whose own name ends in -weft with no owner worktree beside it, expecting it to resolve as itself under the strict anchor check. Add a row for a checkout with no .lym-anchor marker, resolved from a subdirectory, expecting AnchorRel ".".
---

### [MEDIUM] await-forks dispatch and weft-cwd run are untested

**Location:** `internal/shuttlecli/cli.go:87`, tests in `internal/shuttlecli/cli_test.go`.
**Issue:** The change the board entry asks for is that `await-forks` runs from a weft cwd.
The swap `cmd.Name() == awaitForksVerb` is the only place that delivers it, and no test in `internal/shuttlecli` runs the verb from a weft cwd.
A regression that picks `Resolve` for `await-forks`, or swaps the resolver for every verb, fails no test.
`TestResolveWithin` pins the resolver in `lymcwd` but not the dispatch.

### [LOW] ResolveWithin branches without a row

**Location:** `internal/lymcwd/lymcwd_test.go:336`, `internal/lymcwd/lymcwd.go:169`.
**Issue:** `ownerOfRecordsSibling` branches on whether an owner directory sits beside a `-weft` name.
Every `TestResolveWithin` sibling row has its owner present, so removing the existence check passes all rows.
The change description names the no-neighbour case as a design decision.
The no-marker fallback to `.` is also unexercised through `ResolveWithin`; the fixture always records `backend`.

## Rejected

- Code-tests: `ResolveWithin` returning `ErrNotAGitRepo` has no row.
  Rejected: it returns the bare sentinel from `gitWorktreeRoot`, which `Resolve` shares and `TestResolve_NotAGitRepo` covers.
- Code-tests: `..`-prefixed directory name and Windows case folding in `pathWithin` are untested.
  Rejected: the repo tests the public surface, not each helper, and a Windows-only fold cannot run in the Linux gate.
- Code-correctness and code-affected-set returned no findings.
  The handler's own read of the diff, the sentinel text consumers, the docs and `PATTERN-cwd-resolution.md` also found none.
