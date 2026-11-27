# planparser: refuse a new function whose first caller is on a later card

Plan validation now refuses a Go plan in which a card creates a production function that only a later card can call.
Such a card fails its own gate on the guard that every production function is reachable from a `main` (`cmd/lym/testonlycode_linux_test.go`).
Until now the plan passed validation, and the implementer had to add a temporary `testOnlyCodeAllowed` entry to get through.
The later card that had to remove that entry did not list the guard file among its targets.

The new `create-caller-later` finding sits on the creating card and names the first later card that can call the function.
The way forward is to move the `Create` to the card that adds the first production caller, or to list that caller on the creating card.

Closes #273

## Design decisions

- Of the routes the issue offers, this takes plan validation with "declaration and first caller in one card".
  The guard and the card gate are unchanged.
- The check reads plan text alone, in `ValidateFormat`.
- It fires only when the creating card has no ref that could hold a call, and a later card that has one names the handle among its targets or `Uses`.
  A ref whose shape settles nothing, such as a package self glyph or a bare symbol, counts as able to hold a call.
- Another function created on the same card does not count as a caller, unless it lies in a test-support directory.
  On a later card a new function does count, unless it is test code.
- Functions under `internal/testkit`, `internal/gitkit` and `internal/hubforge` are exempt, since the guard counts them as reached.
  `planparser.TestSupportDirectories` is exported so that a `cmd/lym` test can hold it equal to the guard's own list.
- The `TestGlyphChain_RedundantPackage` fixture now creates a type instead of a function, because the new check refuses the old plan shape.

## Limits

- A later card that calls the function without naming its handle is not detected.
- A handle names a package directory, never a file, so a helper in a package-internal `_test.go` file reads as production code.
  A later card that names it and changes production code, or creates another such helper, raises the finding.
  Moving the `Create` to that card clears it.
- A function whose name opens `Test`, `Benchmark`, `Fuzz` or `Example`, or whose head contains `testing.`, is skipped.

## To check by hand

- Group filter and handle-ref branches untested: `internal/planparser/latercaller.go`, `cardCanHoldProductionCaller` and `refCanHoldProductionCaller`.
  `TestValidateFormat_CreateCallerLater` has rows for `Extends`, `Custom`, `Delete` and the handle-ref cases.
  Confirm those are enough; I saw no row for a `Rename`, `Move` or `Prosa` group.
- Test-helper and test-support functions refused: `internal/planparser/latercaller.go`, `isProductionFunctionDeclaration`.
  The test-support directories are now exempt.
  A package-internal `_test.go` helper can still be refused, as under Limits; confirm that is acceptable.
- `TestGlyphChain_RedundantPackage` is a tier 1 test with a changed fixture.
  Run it with `lym gate test --tier tier1 ./internal/planglyph` and check that its cases run rather than skip.

## Merge danger

A run in flight whose plan holds a pending card of the refused shape is refused at its next plan gate or dispatch validation, until the plan moves the `Create`.

**Door:** two-way
**Blast radius:** planning
