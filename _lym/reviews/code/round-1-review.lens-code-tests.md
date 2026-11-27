```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if that behaviour broke. All rows cited are in `internal/planparser/latercaller_test.go`, in `TestValidateFormat_CreateCallerLater`.

**Behaviour → pinning row**

1. The check is skipped outside `go` (`GoToolchain` false): "a plan with no glyph language". `csharp` takes the same branch and has no row.
2. A non-`func` declaration is skipped: "a type, which holds no code to reach".
3. A declaration whose unit is `&lt;dir&gt;_test` is skipped: "a function of an external test package".
4. A name that opens `Test`, `Benchmark`, `Fuzz` or `Example` is skipped: **none**. No row has a test-prefixed name without a `testing.` parameter.
5. An entry function (`main` or `init`) is skipped: "a program entry point" covers `main`. `init` comes from the same slice.
6. A head that names `testing.` is skipped: "a test helper taking a testing type".
7. The creating card can hold a caller:
   - Edit of a member: "the creating card edits a production member that can call it".
   - Create of a `.go` file: "the creating card creates a production file that can call it".
   - A `var` created on the same card: "the creating card creates a variable that can call it".
8. The `caller.Number &lt;= creator.Number` guard: **none**. Every row puts the creator on card 1, and the self case is already masked by `cardCanHoldProductionCaller`.
9. A later card names the handle:
   - In `Uses`: "a function whose only caller is on a later card".
   - In its targets: "a later card that names the function among its targets".
   - Nowhere: "no later card names the function".
10. A later card that cannot hold a caller is skipped: "the later card only changes the function and tests". This row covers the `ref != callee` exclusion and the `_test.go` path. Its test member `internal/watch_test#TestLevel` hits both the `_test` unit check and the test-name prefix check at once, so neither is pinned alone.
11. Only the first later card is reported (`break`): **none**. No row has two later cards that both qualify.
12. Detail text and the `Card` attribution: pinned by the exact-string comparison and the `finding.Card` assertion.
13. The `Ref` field set to the handle: **none**. Nothing asserts `finding.Ref`.
14. Which groups count:
    - Create: pinned (`wire.go` row).
    - Edit: pinned (member row).
    - Extends and Custom: **none**.
    - The default skip of Delete, Rename, Move and Prosa: **none**.
15. How `refCanHoldProductionCaller` treats a handle:
    - A test handle (`isTestHandle`) gives false: **none**.
    - A handle created on another card gives true: **none**.
    - A `func` created here gives false: **none**. The method row relies on its `type` handle.
    - A `type` created here gives false: method row "a method whose only caller is on a later card".
    - A `const` created here gives false: **none**.
    - A `var` created here gives true: variable row.
16. How it treats a disk path:
    - Extensionless and production gives true: the `internal/dobby#` self glyph on the method row's later card.
    - Extensionless with a `_test` suffix gives false: **none**.
    - `.go` versus `_test.go`: pinned by the `wire.go` and `level_test.go` rows.
    - A non-Go extension gives false: row 1 (`template.yaml`).
17. How it treats a parsed member glyph:
    - Production gives true: the `internal/dobby#Tick` rows.
    - `ParseRef` error (bare symbol) gives true: **none**, though the spec states "a bare symbol is one".
    - A `_test.go` file unit gives false: **none**.
18. `declarationKeyword`:
    - Space after the keyword: pinned.
    - Method receiver: "func (w" splits at the space, so it is pinned.
    - A `(` or tab right after `func` (for example `func(`): **none**.
19. The fixture change in `internal/planglyph/glyphchain_integration_test.go` changes no behaviour. The case still creates a handle beside its package self glyph, so it checks what its name states and loosens nothing.

**Findings**

- MEDIUM · design · `internal/planparser/latercaller.go:96-101` — The group-type filter has no row. Removing `CardTypeExtends` or `CardTypeCustom` from the counted list, or dropping the `default: continue` skip for Delete, Rename, Move and Prosa, passes every row. Evidence: `latercaller_test.go:38` defines only `create` and `edit` builders, and no row uses another label.
- MEDIUM · design · `internal/planparser/latercaller.go:116-128` — The handle-ref branches have no row:
  - a test handle on the card gives false;
  - a handle created on another card gives true;
  - a `func` or `const` created on the same card gives false.

  Each mutant survives. Evidence: the only handle refs besides the callee are the `type Watcher` row (`latercaller_test.go:66`) and the `var levels` row (`latercaller_test.go:92`).
- LOW · design · `internal/planparser/latercaller.go:81` — The test-prefix skip (`hasTestPrefix(identifier)`) for a production-package function such as `func ExampleLevel()` has no row. The only test-like row also names `testing.`, so the `testing.` check alone kills the mutant. Evidence: `latercaller_test.go:113`.
- LOW · design · `internal/planparser/latercaller.go:43,61` — Two behaviours have no row: the earlier-card guard (`caller.Number &lt;= creator.Number`), and the `break` that names only the first later card, a behaviour the spec states. No row has a card before the creator, or two later cards that can call the function. Evidence: every row builds the creator as card 1 and has a single caller card (`latercaller_test.go:32-36`).
- LOW · design · `internal/planparser/latercaller.go:130-143` — Four ref shapes have no row:
  - an extensionless path or self glyph with a `_test` suffix gives false;
  - a member glyph whose unit ends in `_test.go` gives false;
  - a `ParseRef` error, the bare symbol the spec says counts, gives true;
  - a member with a `_test` unit or a test name, each condition alone.

  The only test member, `internal/watch_test#TestLevel`, meets both conditions at once (`latercaller_test.go:100`).
- LOW · design · `internal/planparser/latercaller_test.go:156-165` — The finding's `Ref` field (`latercaller.go:52`) is never asserted. Only `Check`, `Card` and `Detail` are checked.

No tool calls made.
