# Card 13 — landing-gates

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify` -> `func (g verifyGate) checkPublishVerify(ctx context.Context, producer, parentBranch string) (string, error)`
- `internal/landingshed#Publish.Call`
- `internal/landingshed#Publish.takeMergeCommitFromRecord`
- `internal/landingshed#MergeIn.Call`
- `internal/landingshed#newPublishVerifyFixture`
- `internal/landingshed#newGateFixture`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/battencli#intentionallyNil`
- `internal/landingshed#TestPublishVerify_Proceeds`
- `internal/landingshed#TestPublishVerify_HaltsOnVerifyResult`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/landingshed#TestPublishVerify_HardErrorsAreReturned`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestFinalizeVerify_Lands`
- `internal/landingshed#TestLoadConfig_OptionalStringKeys`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestMigrateConfig`
- `internal/landingshed/template.yaml`
- `internal/landingshed/doc.go`
- `internal/loomcli#verifySource`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#darnTreeScope`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#writeVerifyPlan`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomcli#seedDarnHubConfig` -> `func seedDarnHubConfig(t *testing.T, hub, verifyAttempts string, replacements ...string)`
- `internal/loomcli#TestPerchHardCap`
- `internal/loomcli#TestWire_LanguageSkills`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#hubLocation`
- `internal/loomcli#hubPublishVerify`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#normalizeTreeScope`
- `internal/darnengine#TreeScopedCommand`
- `internal/darnengine#treeScopeVerifyCommand`
- `internal/darnengine#seedDarnConfig`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine#TestTreeScopedCommand`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#FullEnv`
- `pattern/PATTERN-verified-tree.md`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/recipes/loom-recipe.yaml`
- `docs/overview.md`

**Delete:**
- `internal/darnengine#VerifyCommand`
- `internal/darnengine#verifyWayForward`
- `internal/darnengine#TestVerifyCommand`

**Intent:**
Merge-In, catch-up merge-ins, Finalize and Publish run the hub's `landing` and `publish` gates (D8), and the hand-spelled `publish_verify` and darn `verify` keys retire (D9).

Loom's verify source renders, at each call, from the hub's `gate.yaml` through `gateslot.LoadConfig` over the pair's config dirs: the landing command `GateCall("landing", "--all")`, the publish command `GateCall("publish", "--all")` (`""` for an empty list), and the base command `GateCall("final", "--all")`, which the round gate's told command also reads (card 10) beside the `round` seam card 10 already fills.
Darn's source renders the same landing and publish commands and uses its `landing` call as the round's base command, since darn has no webster gate.
`landingshed.Deps` gains `PublishCommand` and `BaseCommand` readers beside `VerifyCommand`, and the `GateTiers` lookup below; `PublishVerifySkip` becomes `PublishSkip`, which skips the `publish` gate.
Merge-In, catch-up merge-ins and Finalize run the landing call through `verifytree.Verify` with `BaseCommand` set to the base command, so its pass entry survives a landing-list pass where `final` and `landing` differ; two sites with the same list and form render the same command, so Merge-In skips on a tree Webster's gate passed when the two lists match.
Every landing site still passes `verifytree.Verify` today's pool and timeout arguments, which card 14 drops.
Publish runs the landing call, then the publish call, both with the base command as their `BaseCommand` (D8: Merge-In and Publish name that base command on Site), so neither pass prunes the entry the next round diffs from; an empty `publish` list takes the path of today's unset `publish_verify`: nothing runs and the failure record is removed before the push.
A config load or `TierList` refusal is the producer's returned error, carrying the refusal's way forward and role clause (card 2).
A failing landing or publish call writes the Publish failure record with kind `landing` or `publish` and `Tier` from `gateslot.FailedTier` over the failing log; a log without a failing tier summary line, such as a refusal before the first tier, writes the highest tier of the failing gate's list, so the round's confirm pass (card 10) always runs a tier of that gate.
`landingshed.Deps` gains `GateTiers func(gate string) ([]int, error)` beside `PublishCommand`, read at each call, which `landingDeps` fills from `Config.TierList` over the config `gateslot.LoadConfig` loads from the pair's config dirs; `verifyGate.recordFailure` reads it for that fallback tier through a tier-lookup field of `verifyGate`, which `newVerifyGate` sets from `Deps.GateTiers`.
`GateTiers` is required and stays out of `envkit`'s `nilLegal`, which follows only the `PublishSkip` rename, since `landingDeps` always fills it and a nil lookup would panic on a failing log without a summary line; `envkit`'s `LandingDeps` and `newPublishVerifyFixture` fill it over fixed gate lists.
The Stuck reasons name the `landing` or `publish` gate in place of `verify` and `publish_verify`.

A darn run with a tree-scoped diff runs `tree_scope_verify` with `{targets}` filled, in place of the landing call at Merge-In, Finalize and Publish, and skips the `publish` gate, as its skip seam does today.
When the rendered `{targets}` is empty, as for a docs-only tree diff, `TreeScopedCommand` answers scoped with an empty command, and every darn site runs no command, logs it and passes, so an empty list never falls back to the computed scope: the writer gates through card 10's empty tree-scope branch, and Merge-In, Finalize and Publish through the landing gate's existing path for an empty told command, which logs and passes.

`landing.yaml`'s `publish_verify` becomes a retired key (`CategoryExecutable`), dropped at load and pruned by reconcile; its field, template line and doc paragraphs go.
`darn.yaml`'s `verify` retires the same way, with `VerifyCommand`, its way forward and its test; `wireDarn` stops refusing an absent `verify`.
`darnengine.LoadConfig` refuses a non-empty `tree_scope_verify` that is not exactly one `lyx gate test` call holding `--wait` and no shell control operator (`;`, `&`, `|`, a newline, a backtick, `$(`, `<` or `>`), naming `lyx config darn --set tree_scope_verify=<one lyx gate test --wait call holding {targets}>`, so the whole tree-scoped command is slotted and waits out a busy hub; the template's `tree_scope_verify` comment drops its empty-`{targets}` requirement for this rule, and `doc.go`'s example moves from `--tags` to `lyx gate test --wait --tier 1 {targets}`.
Bound: the darn refusal sees static text only, as the shell guard does, and a script the call runs is outside it.

`envkit` fills the renamed and new seams; `PATTERN-verified-tree.md` names the rendered gate calls of every site, the round's base commands and the `publish` gate in place of `publish_verify`;
`orch-template-role.md`'s first-verify-failure sentence, the recipe comment and `docs/overview.md`'s Publish sentence name the `landing` and `publish` gates.

Tests: `TestPublishVerify_Proceeds` covers an empty `publish` list running nothing and removing the record, a darn skip, a darn tree-scoped diff whose command is empty running nothing and passing, and both of Publish's calls reaching the verify seam with the base command as `Site.BaseCommand`; `TestPublishVerify_HaltsOnVerifyResult` and `TestPublishVerify_FailureRecord` assert the gate-named reasons and the record's kind and tier, a failing log without a summary line writing the gate's highest tier included; `TestPublishVerify_HardErrorsAreReturned` uses the renamed `PublishSkip`; `TestMergeIn_PlanVerifyFailureRecord` asserts kind `landing` and the tier; `TestFinalizeVerify_Lands` and `TestLoadConfig_OptionalStringKeys` drop `PublishVerify`; `TestLoadConfig_TemplateDefaults` and `TestMigrateConfig` cover `publish_verify` loading as a retired key.
`TestLandingDeps_VerifyCommandReadsPlanAtCallTime` becomes the rendered calls read at call time from a seeded `gate.yaml`, with a dangling list refused naming `lyx config gate`; `TestLandingDeps_EveryFieldPopulated` checks the renamed and new seams; `TestWire_DefaultConfig`, `hubLocation` and `hubPublishVerify` stop seeding and comparing `publish_verify`, the constant going.
Merge-In's skip on a tree Webster's gate passed rests on two facts each pinned once: `TestConfig_TierUse` (card 2) renders the same call for equal `final` and `landing` lists, and verifytree's existing skip row skips a recorded command on the same tree.
`newGateFixture` hand-builds `verifyGate` and is the gate that `newMergeInFixture` and `newPublishVerifyFixture` install, so it sets the tier lookup over fixed gate lists, since `Deps.GateTiers` set by `newPublishVerifyFixture` never reaches a hand-built gate; `TestMergeIn_PlanVerifyFailureRecord` and `TestPublishVerify_FailureRecord` need it for their logs without a summary line.
`TestWire_Real_PublishRejectedPushNamesRemoteTip` runs over a tierless `hubforge` hub, where `TierList` refuses a `PublishCommand` read for lack of `tier0`, so beside `deps.VerifyCommand = nil` it sets `deps.PublishCommand` and `deps.BaseCommand` to readers returning `""`, the empty `publish` list's path, since no landing site checks either reader for nil.
battencli's `intentionallyNil` gains `Landing.PublishCommand`, `Landing.BaseCommand` and `Landing.GateTiers` with the reason "loom-only seam, batten lands no branch".
`TestDarnTreeScope_ClassifiesAtEachCall` seeds the hub's `gate.yaml` with a `tier0` and a `landing` list, and its non-scoped `assertLanding` rows, which expect darn's retired `verify` value `run all`, assert the rendered landing call `lyx gate test --wait --tier 0 --all`.
`TestDarnTreeScope_ClassifiesAtEachCall` and `TestTreeScopedCommand` cover the empty-targets pass; `TestLoadConfig` of darn gains the `tree_scope_verify` refusals (two calls joined by `&&`, a call without `--wait`, a pipe) and `verify` loading as retired; `TestWire_DarnRecipe` wires without a `verify` key.
Every seeded `tree_scope_verify` the new refusal would reject moves to a `--wait` gate call: darn `TestLoadConfig`'s passing rows that load `lyx gate test {targets}` load `lyx gate test --wait --tier 1 {targets}`, its `tree_scope_verify with no tree` and `tree_scope_verify without targets` refusal rows seed `lyx gate test --wait --tier 1 {targets}` and `lyx gate test --wait --tier 1` so each still meets the refusal it names, and `TestDarnTreeScope_ClassifiesAtEachCall`'s `scopedVerify` constant `run {targets}` becomes `lyx gate test --wait --tier 1 {targets}`.
configcli's `TestDispatchHubWideBoard` seeds and saves a `tree_scope_verify` through the darn editor rows that the new refusal would reject, so its `darnTreeVerify` constant becomes `lyx gate test --wait --tier 1 {targets}`.
`TestWire_DarnRecipe`'s subtest `with tree scope on the verify gate's told command stays the hub's whole verify` seeds `tree_scope_verify: "run {targets}"` through `seedDarnHubConfig`, which the wiring's darn load would now refuse, so its value becomes `lyx gate test --wait --tier 1 {targets}`.
`seedDarnHubConfig` drops its `verify` parameter and its replacement of `verify: ""`, which once the template's `verify` line is gone matches inside `tree_scope_verify: ""` and would write the caller's command there; its callers pass one argument fewer, and `TestWire_DarnRecipe`'s subtest `an empty verify command is refused early with its way forward` goes with the refusal it asserts.

**ImpactSummary:** both landing producers and both recipes' verify sources move to rendered gate calls; two hub keys retire with their readers, old files still load, and old failure records still read through card 10's mapping.
