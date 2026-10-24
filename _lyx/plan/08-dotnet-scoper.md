# Card 8 — the dotnet scoper in lyx gate test

**Create:**
- `plan:internal/gatecli#readCards` -> `func readCards(location *lyxcwd.Location, ids []string) (*planparser.Plan, []planparser.Card, error)`
- `plan:internal/gatecli#dotnetScope` -> `func dotnetScope(ctx context.Context, cfg gateslot.Config, location *lyxcwd.Location, root string, request testRequest, form string, seeds []planparser.CSharpSeed, green *greenRun) (scope, error)`

**Edit:**
- `internal/gatecli#computeScope`
- `internal/gatecli#cardPackages`
- `internal/gatecli#moduleTargets`
- `internal/gatecli#tierScope`
- `internal/gatecli#scope`
- `internal/gatecli#scope.forTier`
- `internal/gatecli#invocation`
- `internal/gatecli#scopeInvocations`
- `internal/gatecli#planTierRuns`
- `internal/gatecli#greenRun.recordPass`
- `internal/gatecli#TestPlanTierRuns`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#scopesConfigWith`
- `plan:internal/dotnetimpact#DeriveDiff`
- `plan:internal/dotnetimpact#DeriveCards`
- `plan:internal/dotnetimpact#Scope`
- `plan:internal/planparser#CardCSharpSeeds`

**Intent:**
Wire the `dotnet` scoper into `computeScope` for the computed, `--base` and `--card` forms (Decisions 1, 3 and 4; Constraint on the green record).

- `readCards` holds the plan read, its two refusals and the id match that `cardPackages` holds today, and `cardPackages` becomes `planparser.CardPackages` over `readCards`' result, so the refusals and their texts stay as they are under every scoper.
- `computeScope` reads the cards under every scoper as today, then under `dotnet` returns `dotnetScope`; the `none` and `go` branches are unchanged.
- `dotnetScope` groups the listed tiers by base, through `diffBaseGroups` for the computed and `--base` forms, and runs `dotnetimpact.DeriveDiff` once per distinct base and tier `dir`, printing `gate test: diff scope from <base> for <tiers>` per derivation as the `go` scoper does; a tier with no base runs whole.
  Under `--card` it runs `dotnetimpact.DeriveCards` once per distinct tier `dir` over `planparser.CardCSharpSeeds` of the cards.
  It passes `cfg.Inert` as the inert predicate.
  A cancelled context is the derivation's one error, returned as the go scoper's derivation error is.
- Each tier's share follows its derivation: `Whole` is a whole share with `gate test: the scope falls back to whole: <reason>` on stderr; `Projects` is one `moduleTargets` per project, in the tier's `dir`, with the project path as its one target and its terms in the new `moduleTargets.terms`; the empty scope is an empty share; and `NoTestClass` sets the new `tierScope.noTests`, which `scope.forTier` copies into the new `scope.noTests`.
  The scope plans no guard step and no vet step, so `--guards` under `dotnet` meets `planTierRuns`' refusal naming the scoper.
  That refusal's way forward is reworded per scoper: under `dotnet` it names only dropping `--guards`, since setting `scoper: go` makes a C# hub's tiers run whole, and under `none` it keeps `drop --guards, or set scoper to go`.
- `scopeInvocations` copies a module's terms into its invocation and sets the new `invocation.partial` on every invocation of a narrowed `dotnet` share, so each selected project runs once in the tier's `dir` under the one slot hold and `timeout_min`, as a narrowed `go` scope runs once per module.
  A command without `{targets}` still runs once, whole, with its tier filter only.
  Under `noTests`, a tier with `shared_build` gets no invocation, and any other tier runs whole; `planTierRuns` prints `gate test: the scope selected no test class, so <tierN> runs whole` for it on stderr, whether or not another listed tier opts in, so a compile break still fails the gate (Decision 3); card 11 adds the build step to the opted-in tier.
- `greenRun.recordPass` writes no entry for a `partial` invocation, even when its `{targets}` equal the tier's `all_targets`, so a filtered or project-narrowed pass never stands in for a whole run; a whole `dotnet` pass writes its whole entry as under `none`, and only whole entries carry, since `openGreen` is told the scoper is not `go`.

`doc.go`'s Scoper section gains the `dotnet` scoper: its derivation per base and `dir`, the per-project invocations with `{targets}` and `{filter}`, the two empty scopes, and its green record rule, and its sentence that the `none` scoper's forms are whole and `--guards` is refused under it is rewritten to cover `dotnet`.

Tests:
`TestPlanTierRuns` gains rows over hand-built `dotnet` scopes, with the `describe` helper also rendering an invocation's terms and `partial`:
a narrowed share running one invocation per project with its terms and `partial` set;
a narrowed share for a tier whose command lacks `{targets}`, running once whole with no terms;
a scope whose base groups split, one tier's share narrowed and another's whole, the whole tier running its `all_targets` with no terms;
a `noTests` share giving an opted-in tier no invocation and running another whole with the stderr line;
and `--guards` under `scoper: dotnet` refused with the exact message naming the scoper and only dropping `--guards`.
`TestGateTest_Scopes` gains `dotnet` rows over a small C# tree added to the seed commit (`cs/Lib/Circle.cs`, `cs/Lib.Tests/Lib.Tests.csproj` with a `CircleTests` class naming `Circle` and an `OtherTests` class), each switching the scoper with `scopesConfigWith` and the stand-in command `{targets} {filter}`.
The computed-scope rows sit before the rows that commit to the pair, beside the Markdown-only row, so the merge base the pair diffs from holds no other change:
a computed scope over an edit to `Circle.cs` runs `cs/Lib.Tests/Lib.Tests.csproj` once with `--filter FullyQualifiedName~…CircleTests`;
a `.csproj` edit falls back to whole, naming the path;
and a `--card` row over a `language: csharp` plan, written with `plankit.Write` and restored to the Go plan after, whose target is a `Circle` member glyph runs the same project and term.
The `--card` refusal loop over scopers gains `dotnet`, so the unknown-id refusal is held under it.
After the committing rows, on a clean tree:
`Circle.cs` is changed and committed, `--base` at the prior commit runs the same project and term, and the green record then holds no entry for the tier, with its `all_targets` set to that one project, so the row fails if a narrowed pass is recorded; a later `--all` runs the tier;
and two tiers of `counter: vstest` both passing `--all` carry at a second `--all` on the same tree.

**ImpactSummary:** `computeScope` and `scopeInvocations` gain a `dotnet` branch beside unchanged `go` and `none` branches; `cardPackages` keeps its signature and refusals, and the new fields are zero on every path but the `dotnet` one.
