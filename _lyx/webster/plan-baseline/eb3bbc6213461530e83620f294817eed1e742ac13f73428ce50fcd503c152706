# Card 9 — gate-scopes

**Create:**
- `plan:internal/gatecli#scope` -> `type scope struct`
- `plan:internal/gatecli#computeScope` -> `func computeScope(ctx context.Context, cfg gateslot.Config, location *lyxcwd.Location, root string, request testRequest) (scope, error)`
- `plan:internal/gatecli#parentMergeBase` -> `func parentMergeBase(location *lyxcwd.Location) (string, bool, error)`
- `plan:internal/gatecli#TestGateTest_Scopes` -> `func TestGateTest_Scopes(t *testing.T)`

**Edit:**
- `plan:internal/gatecli#planTierRuns`
- `internal/gatecli#runTest`
- `plan:internal/gatecli#TestPlanTierRuns`
- `internal/gatecli/doc.go`

**Uses:**
- `plan:internal/impactset#DeriveTargets`
- `plan:internal/planparser#CardPackages`
- `internal/planparser#ParsePlan`
- `internal/planparser#PlanDir`
- `plan:internal/gatecli#runGuardStep`
- `internal/gatecli#startGate`
- `internal/gatecli#runGate`
- `internal/hubforge#CopyHub`

**Intent:**
Wire the computed, `--base` and `--card` scope forms (D2, D3, D4, D5).

`computeScope` runs inside the pre-tier ceiling, passing the ceiling's context to `impactset.DeriveTargets` so an expiry kills a stalled `go list`, and returns the run's scope: whole, or per-module target lists, plus the guard packages for the guard step and a label for the summary line and the run record.
- Computed (no scope flag, no targets): the base is the merge base of HEAD with the parent branch the pair's fabric origin record names, read through `parentMergeBase` with `fabricengine.ReadOrigin` and the fabric handle's `MergeBase`; a worktree with no origin record or no parent, such as the prime on `main` or a standalone run, runs whole.
- `--base <rev>`: that revision is the base.
- `--card <id>`, repeatable: the plan is read through `planparser.ParsePlan` at the anchor's plan directory, and the targets are `planparser.CardPackages` over the named cards; an unknown id is refused before any slot naming the plan's card ids, and a worktree with no plan is refused naming `--all` or explicit targets.
- Under the `none` scoper each of the three is whole, `{targets}` the tier's `all_targets`; the `--card` plan read and its two refusals run first under either scoper, so an unknown id is refused under `none` too.
- Under the `go` scoper computed and `--base` take `impactset.DeriveTargets` against the base over the working tree, and fall back to whole wherever it does, naming the reason on stderr; `--card` takes the cards' packages per module.
A narrower `go` scope runs each listed tier once per module, in that module's directory with its module-relative targets, under one slot hold and one `timeout_min` covering every invocation (card 8).
An empty computed, `--base` or card scope runs no listed tier's command: each listed tier passes with zero discovered and is logged so, with no slot taken, and only the guard step runs.
Under the `go` scoper a computed, `--base` or card scope then runs the guard step (card 8's `runGuardStep`) over the module's `//lyx:guard` tests outside the scope, whose packages `DeriveTargets` names for a narrower computed or `--base` scope and a `ModuleGuardTests` scan names for a card scope or a computed or `--base` scope fallen back to whole, so the round and darn writer gates, which run `--base` (card 10), keep the guard coverage they have today, where every guard package counts as outside the tiers that do not compile it; the step is skipped only when the call ran `guard_tier`, or a tier covering it, whole (D5).
A guard scan error refuses the call, before any slot, with its file, line and way forward.
`--all` only widens.

`runTest` resolves the scope before `planTierRuns`, which takes it as the run list's targets; the card-8 refusal of `--card` and `--base` goes.

Bound: `--base`, `--card` and the computed form change only which targets a tier runs over and never the slot rules, and add no tier but the guard step's run of `guard_tier`; the `go` scoper narrows only where `impactset` narrows today, plus the empty set, whose skipped `go build ./...` and whatever the import graph cannot see is left to the whole runs at `final` and `landing`.

`doc.go` documents the scope forms, the scopers, the per-module runs, the empty set and the guard step.

Tests: `TestPlanTierRuns` gains rows for a per-module scope, an empty scope running no command, and a whole fallback with its guard step still planned for `card: [0]` and `guard_tier: 1`.
The new `TestGateTest_Scopes` drives the built verb over a `hubforge` hub with a task pair whose fabric origin record names a parent, seeded with a small Go module, a plan and a stand-in tier command; the stand-in scenario test's hub is the prime alone, with no origin record and no parent, so it cannot reach a computed scope narrower than whole.
Its rows: a computed scope under `go` running the changed package and its importer and then the guard step through `guard_tier`'s slots and timeout; a computed scope over a change to a nested module's file falling back to whole, naming the reason on stderr; an `.md`-only change running no tier command, logging zero and running only the guard step; a `--base` run of the changed package, followed by the guard step through `guard_tier`'s slots; a `--card` run over a card whose targets span the root module and a nested module, which `CardPackages` groups per module (card 6), running the tier once per module, with a `timeout_min` tier killed across the two invocations and the timeout line; a `--card` run of a card's packages with its guard step; an unknown card id refused before any slot naming the ids, under `go` and under `none`; a worktree with no plan refused naming `--all`; a misplaced `//lyx:guard` marker refusing a computed call before any slot with its file and line; the prime with no parent running whole; and, under `none`, a computed run taking `all_targets`, a `--card` run taking `all_targets`, an `.md`-only diff running the listed tiers whole with a covered tier skipped, and `--base` taking `all_targets`.

**ImpactSummary:** the verb gains its remaining scope forms; no caller exists yet beyond agents and the stencils cards 10 to 16 move onto it.
