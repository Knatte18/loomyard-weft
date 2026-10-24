# Round 1 fixer-report — plan

Every finding's premise held against the code at HEAD and `_lyx/discussion/decision-record.md`; all thirteen are fixed.

## Changes

- **F1** (MEDIUM), as proposed: card 10's Narrowed gates bullet says that under `go` an inert-only group counts as narrowed and leaves the guard tests to the module scan, so an all-inert call returns the `anyWhole`-branch scope (`guardStep` set, `guardDir` `tier0`'s dir, `guards` nil), never the `!narrowed` whole return nor `guardsByTier(nil)`; under `none` the early whole return becomes a per-group share, empty only for an inert-only group, with `guardStep` false.
- **F2** (MEDIUM), as proposed: card 8's `TestGateTest_Scopes` paragraph names the full seed tree, adding `cs/Lib/Lib.csproj`, `cs/Lib/Unused.cs` (type `Unused` named by no test class) and the test files `CircleTests.cs` and `OtherTests.cs`; card 11's no-test-class row edits `cs/Lib/Unused.cs` from card 8's tree.
- **F3** (LOW), as proposed: card 8's `--guards` bullet says the `dotnet` refusal is not built through `configRefusal` and gives its exact text; `none` keeps its `configRefusal` text; the `TestPlanTierRuns` row asserts the exact `dotnet` text.
- **F4** (LOW), as proposed: card 6's line on failed git reads says a failed `git diff`, `git ls-files` or `git show`, apart from a path absent at the base, returns `Whole` naming the git error, and why no row forces one.
- **F5** (LOW), as proposed: card 11's `runBuild` bullet refuses a build whose shell cannot start, naming its `dir` and the `make build.dir an existing directory …` way forward; `TestGateTest_Scenario` gains a missing-`dir` row.
- **F6** (LOW), as proposed: card 2's `verifyGate.verdict` bullet opens a build timeout's reason with `fix the hanging build, such as a stalled restore or compile, on the task branch`; the `TestVerifyGate_CheckMapsVerifyResult` build row asserts it.
- **F7** (LOW), as proposed: card 2 adds `RunRecord.BlockedBy` (`json:"blocked_by,omitempty"`), names it in the package doc bullet and adds a `TestWriteRunRecord` round-trip row; card 11's `reportBlocked` sets it, and the build-exits-3 row asserts it names the build's log.
- **F8** (LOW), fixed differently: card 8's Intent states what the narrowing skips and its Bound, but leaves the misses to the gates that run whole (Decision 10). It does not say they are bounded by Decision 3's fallbacks, as the proposal had it, because `decision-record.md:70` says the fallbacks cover only what the derivation can detect and leaves the Bound's misses to Decision 10. The fix-claims self-check raised this.
- **F9** (LOW), as proposed: card 8's `scopeInvocations` bullet sets `partial` only on the per-project invocations; the `TestPlanTierRuns` row without `{targets}` asserts `partial` unset.
- **F10** (LOW), as proposed: card 11's Edit list gains `internal/gatecli#newCommand`, and a bullet adds the build step and the `--guards` refusal under `none` and `dotnet` to its Long help.
- **F11** (NIT), as proposed: card 1's scoper bullet names `Config.Scoper`'s comment as `go`, `none` or `dotnet`.
- **F12** (NIT), as proposed: card 7 wraps a quoting error of the tier's `Filter` in a refusal naming the tier and the way forward `fix the tier's filter with "lyx config gate" from the prime`, with the role clause.
- **F13** (NIT), as proposed, taking the pass-the-number option: the budget compares the whole composed expression against `greenCommandBudget`; `filterArgument` and `renderTierCommand` take the tier number, which `runInvocations` passes as `run.number`, so both the Info log and F12's refusal can name the tier.

## Self-check

- fix-affected-set, LOW, `planTierRuns`' doc comment still lists `--guards` among the `lyx config gate` refusals for `none` only: fixed as proposed, card 8 rewrites that comment.
- fix-affected-set, LOW, `computeScope`'s doc comment still says every form is whole under `none`: fixed as proposed, card 10's doc bullet rewrites it beside `doc.go`'s twin.
- fix-claims, MEDIUM, card 8's new Bound sentence credits Decision 3's fallbacks with catching the closure's misses: fixed as proposed, the sentence now leaves the misses to Decision 10's whole gates.
- fix-consistency, LOW, card 7's ImpactSummary and `TestRenderTierCommand` paragraph do not mention the new tier-number parameter: fixed as proposed.
- fix-proposal, MEDIUM, card 11 does not say whether a build that cannot start halts the call or blocks the opted-in tiers: fixed as proposed. The refusal travels in `tierResult.refusal`, so it halts the call with exit 1 as a tier refusal does (`internal/gatecli/run.go:418`), writes no build record and reports no blocked tier; the missing-`dir` row asserts exit 1 and that no tier command runs.
- fix-proposal, LOW, card 6's git-read `Whole` sentence conflicts with the cancelled `ctx` being the one error: fixed as proposed, a failed read while `ctx` is done returns `ctx`'s error.

## Disputed

None.

## Deferred

None.
