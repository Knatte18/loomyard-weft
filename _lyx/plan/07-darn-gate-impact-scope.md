# Card 7 — darn-gate-impact-scope

**Create:**
- `plan:internal/loomcli#darnTreeScope` -> `func darnTreeScope(l *lyxcwd.Location) func() (string, bool, error)`
- `plan:internal/loomshed#roundGateFixture.diffGate` -> `func (f roundGateFixture) diffGate(command, mergeBase func() (string, error), treeScope func() (string, bool, error)) (shuttleengine.GateResult, error)`
- `plan:internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall` -> `func TestDarnTreeScope_ClassifiesAtEachCall(t *testing.T)`

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), treeScope func() (string, bool, error), slots *gateslot.Pool) shuttleengine.Gate`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolvePlainGateClosure`
- `contracts/specs/shed-recipe-spec.md`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomengine#RecipeOverride`
- `internal/loomengine#TestRecipeOverride`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/loomcli#TestWire_DarnRecipe`

**Delete:**
- `internal/loomshed#roundGateFixture.wholeDiffGate`

**Uses:**
- `internal/loomcli#pairMergeBase`
- `internal/loomcli#seedDarnHubConfig`
- `internal/darnengine#LoadConfig`
- `internal/darnengine#VerifyCommand`
- `plan:internal/darnengine#TreeScopedCommand`
- `plan:internal/darnengine#ErrTreeScopeKey`
- `plan:internal/darnengine#WithEditorWayForward`
- `internal/verifytree#Verify`
- `plan:internal/fabricengine#Fabric.ChangedPathsBothSides`
- `internal/impactset#Derive`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/gitkit#CommitFile`

**Intent:** Replace the whole-diff form of darn's Darn-Write and Darn-Burler-Fix verify gates with D3's impact-scoped form, told D2's tree classification.

`NewVerifyGate`'s `wholeDiff bool` becomes `treeScope func() (string, bool, error)`: nil keeps the round form loom's Webster-Burler-Fix and Merge-Burler-Fix gates run, unchanged; non-nil selects the darn form, which the wiring tells only for a darn run.
In the darn form, at each arrival, the gate:
1. reads the told command, the hub's whole `verify`, and returns an error for an empty one or a read error, and an error for a nil `mergeBase`, as the whole-diff form did;
2. reads the merge base and runs the headroom lint and then the comment lint from it to HEAD;
3. reads `treeScope`; a read error is the gate's returned error;
4. when the diff is tree-scoped, runs the tree-scoped command with no trailing `tmux` package pass;
5. otherwise runs `impactset.Derive(worktreeRoot, mergeBase)`, so every attempt diffs from the merge base and a PR-Gate rework is never narrowed to a stale pass: a guard-scan rejection fails with its findings as in the round form, a derived command runs with today's `tmux` pass over its packages for a `publish_verify` record, and a fallback runs the told command with today's `./...` `tmux` pass for such a record;
6. replays a checked Publish failure record's failing tests by name ahead of that command, as today.
The told command stays the site's `BaseCommand`, so a pass keeps its record entry.
The function's doc, `verifygate.go`'s header line and `doc.go`'s verify gate paragraphs describe the two forms; D3's bound, that an impact-scoped pass lets through a break only the whole module shows and Merge-In's whole run catches it for every diff touching production code, goes in `doc.go`.

`Env.VerifyWholeDiff` is removed and `Env.VerifyTreeScope func() (string, bool, error)` added, read by the `verify` gate and requiring `VerifyMergeBase` when set; `resolveGateSpec` passes it and its doc names it, and `shed-recipe-spec.md`'s two `verify` sentences say so.
`nilLegal` gains `VerifyTreeScope`, since a loom run leaves it nil.

`darnTreeScope(l)` returns the seam: each call loads `darn.yaml` through `darnengine.LoadConfig` over `fabricengine.ConfigDirs(l)`, answers `("", false, nil)` when tree scope is off, and otherwise reads the merge base through `pairMergeBase(l)`, the task diff through `Fabric.ChangedPathsBothSides(mergeBase, "HEAD")` and returns `darnengine.TreeScopedCommand` over them; every error is returned.
`wireDarn` sets `c.env.VerifyCommand` to a closure over `darnengine.VerifyCommand`, so the gate's told command stays the whole `verify`, sets `c.env.VerifyTreeScope = darnTreeScope(location)`, and drops `VerifyWholeDiff`.
Its run-start load refusal returns, for an error wrapping `darnengine.ErrTreeScopeKey`, `darnengine.WithEditorWayForward`'s error instead of appending `hubConfigWayForward`; the input that reaches it is a hub `darn.yaml` holding `./tools` under `tree_scope`.
`lyx loom start` loads `darn.yaml` earlier, at the seed stage, through `loomengine.RecipeOverride`, as do the drive wiring's tackler read and batten's arm, so `RecipeOverride`'s darn branch returns its load error through `darnengine.WithEditorWayForward` too, and the editor way forward reaches the operator from every run-start reader.
`wireDarn`'s doc comment names the editor way forward for a malformed tree key, and `darnwiring.go`'s header comment drops the whole-diff form.

`TestVerifyGate_Scenario` and `TestGateParity_VerifyGate` pass a nil tree scope where they passed `false`, and `runGate` likewise.
`wholeDiffGate` is replaced by `diffGate`, and `TestVerifyGate_RoundScenario`'s whole-diff steps become darn-form steps over the fixture, each told the merge base `planPass.Commit`:
- a tree-scoped answer runs only the told scoped command, records its pass under that command with the told command's entry kept, and the told command does not run;
- right after that pass, with no failure record present, `verifytree.Verify` at site `Merge-In` with the same scoped command on the same tree answers `StatusSkipped`, so a Merge-In after a writer-gate pass skips through the record;
- a tree-scoped answer with a `publish_verify` failure record runs the failing tests and the scoped command with no `go test -tags tmux` step;
- a not-scoped answer after a change to `b/b.go`, told as merge base the commit just before that change while the told command's latest recorded pass is the older `planPass.Commit` with later changes to `a/a.go` between them, records a pass of the command `impactset.Derive` gives from the told merge base and none of the different command it gives from `planPass.Commit`, with the `tmux` pass over its packages for a `publish_verify` record;
- a not-scoped answer after a change to a non-Go, non-Markdown file outside any package runs the told command in full, with `./...` under `tmux` for a `publish_verify` record;
- a command read error, an empty command, a merge base read error, a nil merge base reader and a tree scope read error each return an error and run nothing;
- the misplaced-marker step's darn-form call, told a not-scoped answer, fails with the same `a/a_test.go:5` guard-scan findings as the round form;
- the plan-verify-record step's darn-form call, told a not-scoped answer, records the failing tests followed by the derived command with nothing after it;
- the comment-lint step's darn-form call fails naming `a/a.go:3` from the merge base;
- the headroom step's darn-form call, told the parent commit as merge base, fails on the grown overview before any command runs.

`TestWire_DarnRecipe` expects `VerifyTreeScope` set for a darn seed and nil for a loom seed instead of the `VerifyWholeDiff` flag, `c.env.VerifyCommand()` returning the hub's `verify` even with tree scope on, and gains a subtest: a hub `darn.yaml` with tree key `./tools` makes `wire` fail naming `./tools` and `lyx config darn`'s editor.
`TestRecipeOverride` gains a row: a hub `darn.yaml` holding `./tools` under `tree_scope` makes the darn recipe's read fail naming `./tools` and `lyx config darn`'s editor.

`TestDarnTreeScope_ClassifiesAtEachCall`, in an `integration`-tagged file, builds a hub with `hubforge.CopyHub` and a task pair with `hubforge.AddPair`, seeds the hub `darn.yaml` with `seedDarnHubConfig` holding `docs: ""` and `tools: ./tools/...` and a `tree_scope_verify` of `run {targets}`, and asserts in steps over the pair's code worktree:
- with no commit the seam answers `run ` and true;
- after a commit to `docs/ä.md` and `tools/a.go` it answers `run ./tools/...` and true;
- after the parent branch gains a commit to `internal/p.go` and the task merges it with no change of its own, it still answers `run ./tools/...` and true, the diff read from the merge base the merge left;
- after the parent gains another commit and the task merges it with a resolution that adds `internal/r.go`, it answers false;
- on a fresh pair, a commit renaming a committed `internal/x.go` to `tools/x.go` answers false, the old path counting as production;
- with the hub `darn.yaml` holding `./tools` under `tree_scope`, the seam returns an error naming `./tools`;
- with `tree_scope_verify` emptied it answers false.
Its merge-base and diff reads propagate raw git errors, which the refusal spec exempts as transient, and take no row.

**ImpactSummary:** the verify gate constructor is re-signed for its four callers; loom's round form and every loom wiring are unchanged, darn's two gates trade the whole-diff run for the impact-scoped or tree-scoped command, and `RecipeOverride`'s darn branch gains only a way forward on one refusal.
