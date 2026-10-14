# Card 3 — Export impactset's guard scan and command limit

**Rename:**
- `internal/impactset#findGuardTests` -> `plan:internal/impactset#FindGuardTests`
- `internal/impactset#maxCommandLength` -> `plan:internal/impactset#MaxCommandLength`

**Edit:**
- `internal/impactset#GuardScanError`
- `internal/impactset#guardNamesInFile`
- `internal/impactset#Derive`
- `internal/impactset#commandFor`
- `internal/impactset#TestFindGuardTests`
- `internal/impactset/doc.go`

**Intent:**
Make the guard-test marker scan callable by webster, so the round gate and the card gate share one scan (decision record, Scope and D2), and expose the command-line limit D2's over-long rule reuses.

Rename `findGuardTests` to `FindGuardTests` and `maxCommandLength` to `MaxCommandLength`, retargeting their uses in `Derive` and `commandFor` and the call in `TestFindGuardTests`; behavior is unchanged.
`FindGuardTests`'s doc comment adds that the caller passes the package directories, worktree-relative and slash-separated: `Derive` passes `go list`'s, and webster passes its own walked list.

Add a `File string` field to `GuardScanError`: the worktree-relative, slash-separated path of the test file the error sits in.
`guardNamesInFile` sets it to `rel` in every `GuardScanError` it returns, so webster can tell whether the file lies in a package a card targets (D3).
The message is unchanged.

`TestFindGuardTests` asserts, for every error row, that `errors.As` yields a `*GuardScanError` whose `File` is `dir/p/p_test.go`.
The impactset round gate's guard tests stay as they are; the existing `TestCommandFor` and `TestDerive_OverARepository` keep pinning them.

`internal/impactset/doc.go` adds one sentence: the marker scan is exported as `FindGuardTests`, and webster's card gate runs it over its own walk of the root module.

**ImpactSummary:** A rename of one function and one constant inside `internal/impactset` with no caller outside it, plus a new error field; `Derive`'s command is unchanged.

**Commit:** 3: Export impactset's guard scan and command limit
