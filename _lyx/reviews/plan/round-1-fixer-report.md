# Plan round 1 — fixer report

## Fixed

### F1 — card 5's untagged-tier seam leaves another package's test spawning `go list`

Validated: `internal/webstercli/cli_test.go` is untagged, and `TestValidateCmd_Envelopes` writes `deleteOrderPlan` (Delete `sub#Foo`) over `plankit.Repo(t, deleteOrderFiles)`, whose `sub/b.go` calls `Foo` and whose tree holds no `go.mod`.
`plankit.Repo` writes only the given files.
A scan of the other packages' untagged tests found none that pairs a `go.mod` with a delete or re-sign through a plan gate.

Change, card 5:
- `callerCoverageFindings` skips the load without calling any loader when the told worktree root holds no `go.mod`, so D2's fallback answers it.
- The "grep before landing" sentence is replaced by the statement of what this protects, naming `TestValidateCmd_Envelopes`, `deleteOrderPlan` and `deleteOrderFiles`, and that no other package's untagged test runs a plan gate over a delete or re-sign in a tree holding a `go.mod`.
- `TestGlyphChain_CallerCoverageTyped` gains a row: a fixture copy with its root `go.mod` removed, whose loader fails the test when called, answered with the scan's findings.
- The docs sentence adds the skip.

### F2 — the per-identifier fallback has no mechanism

Validated: the re-signed `scanReferences` takes whole files, so passing a checked file re-reports every typed non-reference in it.

Change, card 5:
- `typedReferences`' Create head gains a third result, `unresolved []map[string][]int`, indexed like `subjects`: the file and line of each identifier in a checked file named like a subject with neither a `Uses` nor a `Defs` object.
- The scan runs over unchecked candidates plus checked files holding an `unresolved` line, and a scan match in a checked file is kept only at a line `unresolved` records for that subject.
- `TestGlyphChain_CallerCoverageTyped` gains a row: a type-checked file holding `undefined().Method()` and a typed `u.Method()`, the first reported through the scan at its line only, the second not reported.
- I also fixed an inconsistency in the same card: the row text named the other type's method `M` and the call `u.M()`. A method named `M` is not named like `callees#Thing.Method`, so the row could never test what it claimed. It now says `Method` and `u.Method()`.

### F3 — card 4 does not bound the import-selector exclusion

Change, card 4:
- A bound paragraph after the `referencesSubject` rules says the exclusion also drops a method call on a local name that shadows an imported package's name (`errors.M()` after `errors := …`), even informationally.
  It also says the exclusion is bounded to the scan: every file until card 5 lands, and afterwards only a file or identifier without type information.
- The spec-row docs sentence carries that bound.
- Card 5's docs sentence narrows that bound to the fallback.

`lyx loom validate-plan` passes after the changes, with only the informational `create-new-unit` findings.
None of these changes adds a top-level test, so no budget row changes.

## Disputed

None.

## Deferred

None.
