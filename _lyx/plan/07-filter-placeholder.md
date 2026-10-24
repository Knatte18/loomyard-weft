# Card 7 — the {filter} placeholder and the tier filter

**Create:**
- `plan:internal/gatecli#filterArgument` -> `func filterArgument(tier gateslot.Tier, terms []string) (string, error)`

**Edit:**
- `internal/gatecli#invocation`
- `internal/gatecli#renderTierCommand` -> `func renderTierCommand(tier gateslot.Tier, targets, terms, args []string) (string, error)`
- `internal/gatecli#runInvocations`
- `internal/gatecli#TestRenderTierCommand`
- `internal/gatecli/doc.go`

**Intent:**
Render a tier's `{filter}` under every scoper, keyed on each invocation's own share of the scope (Decision 4 and the parent's added decision on the bound).

`invocation` gains `terms []string`, the class filter terms of an invocation the `dotnet` scoper narrowed to one selected project; it is nil for every other invocation: a whole run, explicit targets, darn's tree-scope targets, a `go` scope, the vet and guard steps.
`renderTierCommand` takes the terms and fills `{filter}` with `filterArgument`'s result; `runInvocations` passes `invocation.terms`, so every existing caller's invocations render the tier filter alone.

`filterArgument` returns `--filter ` followed by the expression quoted for the platform shell by `verifyrun.QuoteArg`, or "" when there is no expression:

- with no terms, the expression is the tier's `Filter`, and none when the tier has no filter;
- with terms, it is `(<tier filter>)&(<term>|<term>...)`, or `<term>|<term>...` when the tier has no filter;
- when the terms joined by `|` exceed `greenCommandBudget`, the bound a carried-over remainder already uses, the expression is the tier's `Filter` alone, so that project runs whole within itself while staying narrowed to it, and an Info log names the tier and the dropped term count.

A quoting error is returned as `renderTierCommand`'s.
`doc.go`'s sentence listing a tier's placeholders is rewritten to add `{filter}`, and it names its two renderings and the budget fallback, and the Bound of Decision 4: a tier's `filter` applies at whole scope too, landing gates included, bounded by the executable, hub-file-only `tiers` key.

Tests:
`TestRenderTierCommand` gains a `terms` column and the tier's filter, with rows: a tier filter alone with no terms (`--filter 'Category!=Slow'`); no tier filter and no terms rendering nothing; terms alone (`--filter 'FullyQualifiedName~A|FullyQualifiedName~B'`); both composed (`'(Category!=Slow)&(FullyQualifiedName~A)'`); terms past `greenCommandBudget` rendering the tier filter alone; and terms given to a command without `{filter}`, which renders no filter, so a `dotnet` tier without the placeholder runs each selected project with no class filter.

**ImpactSummary:** `renderTierCommand` has one production caller, `runInvocations`, re-signed here with nil terms for every invocation today; a command without `{filter}` renders unchanged.
