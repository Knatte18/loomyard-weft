# Card 6 — planparser-card-packages

**Create:**
- `plan:internal/planparser#ModulePackages` -> `type ModulePackages struct`
- `plan:internal/planparser#CardPackages` -> `func CardPackages(plan *Plan, cards []Card, worktree string) []ModulePackages`
- `plan:internal/planparser_test#TestCardPackages` -> `func TestCardPackages(t *testing.T)`

**Edit:**
- `internal/websterengine#gateScopeOf`

**Delete:**
- `internal/websterengine#appendUnique`
- `internal/websterengine#moduleRelative`
- `internal/websterengine#holdsGoFile`

**Uses:**
- `internal/planparser#CardTargetDirs`
- `internal/planparser#NestedModules`
- `internal/planparser#ModuleOf`

**Intent:**
Move the card-scope rule Webster's card gate applies today into `internal/planparser`, so `lyx gate test --card` (card 9) computes the same packages through the plan's sole parser (`PATTERN-sole-parsers`).

`CardPackages(plan, cards, worktree)` returns the Go package arguments the cards' targets live in, grouped per module, the root module (`Dir` `.`) first when it has any and each nested module in first-seen order, each package once and module-relative.
The rule is `gateScopeOf`'s unchanged: a directory drops out when every target in it is a `Delete`, or when it neither names Go source nor directly holds a `.go` file in `worktree`; a directory belongs to a nested module as the plan stands once the last of the cards has run.
`ModulePackages` holds `Dir` and `Packages`.

`gateScopeOf` now builds its `gateScope` from `CardPackages`, so the webster card and batch gates keep their rendered steps byte for byte until card 15 replaces them; the three helpers only it used move into planparser with the rule and are deleted here.

Tests: the new `TestCardPackages` covers the public rule: a root-module package, a nested module's module-relative package, a Delete-only directory and a stencil-only directory dropped, a repeated directory listed once, and an empty card list.
`TestCardGateCommand` keeps covering the webster rendering unchanged.

**ImpactSummary:** one webster helper now delegates and three unexported helpers move out; the rendered card and batch gates are unchanged.
