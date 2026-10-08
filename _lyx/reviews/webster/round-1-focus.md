---
round: 1
focus:
  - "Card 5, `internal/planglyph`: settle whether the types load is bounded, logged and offline as the card states, and whether every failure path falls back to the import-path scan rather than dropping a reference."
  - "Card 5, `TestMain` and `defaultTypesLoader`: settle whether any untagged test can spawn `go list`, and whether the loader's timeout is a field rather than a package-level duration."
  - "Card 4, `importPathOf` and the caller scan: settle whether name-only method matches collapse correctly, and whether nested modules are reached through the on-disk set with `through` 0."
  - "Cards 1 to 3, `planparser.NestedModules` and `ModuleOf`: settle whether the card gate and `verify-nested-module` derive nested modules only through them, and whether a Verify value that tests a nested module from outside it is refused while the `go -C <module>` form passes."
  - "Per-card mechanical check: for each card with a `Rename` or `Delete` group, settle whether the type-specific check ran against every target of that group, not only the first."
  - "Test economy: for each new top-level test in cards 1 to 5, settle whether an existing test already covers the behavior, naming that test, and whether each added behavior has coverage."
  - "Card 7 and the test budget: settle whether the new tests keep each package within its row in `cmd/lyx/testdata/test-budget.yaml`, and whether the file is edited where a package passes its row."
  - "Docs landing: settle whether the new check and the caller scan change are reflected in the package `doc.go` files, `docs/overview.md` and `PATTERN.md` in the same commit where a rule requires it."
---

Seed focus from the plan overview only; no diff was read.
