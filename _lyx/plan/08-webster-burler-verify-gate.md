# Card 8 — Webster-Burler carries a `verify` gate

**Create:**
- `internal/loomshed/verifygate.go`
- `internal/loomshed/testmain_test.go`
- `internal/loomshed/verifygate_integration_test.go`

**Edit:**
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_gate_test.go`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomcli/wiring.go`
- `internal/testkit/envkit/envkit.go`

**Uses:**
- `internal/verifytree/verify.go`
- `internal/loomshed/gates.go`

**Intent:**
D8: a Burler round that changed code ends with a passing verify.

`loomshed.NewVerifyGate(anchorPath, worktreeRoot, verifyDir, siteLabel string) shuttleengine.Gate` (new `verifygate.go`, beside `NewPlanGate` in `gates.go`) returns a must-pass closure that, at each arrival, parses the plan under `anchorPath` for its `## verify:` command and calls `verifytree.Verify` in `worktreeRoot` with `verifytree.NewPaths(worktreeRoot, verifyDir)`, the site label and the closure's own count of its calls as the attempt.
`StatusPassed` and `StatusSkipped` pass;
`StatusDirty` fails with the dirty paths;
`StatusFailed` fails with the exit code, the log path and the log's tail;
a plan with no `## verify:` section passes with a logged warning;
a plan read error or a cancelled verify is a returned error.
It is exported, like the other plain gates, so the parity test (card 18) drives the same closure the recipe resolves.

`shedrecipe.Env` gains `VerifyDir`, filled by `loomcli` wiring with `verifytree.Dir(anchorPath)` and by `envkit` with an absolute path under its fixture dir.
`resolvePlainGateClosure`'s closed vocabulary gains `verify`:
it requires absolute `Env.AnchorPath`, `Env.WorktreeRoot` and `Env.VerifyDir`, and returns `loomshed.NewVerifyGate` over them with site label `<row name> gate` (`Webster-Burler gate` on the shipped row).
The vocabulary error message and `resolveGateSpec`'s doc name all six values.

`loom-recipe.yaml`'s Webster-Burler row gains `gates: [{name: verify, attempts: 3}]`, replacing its "No gates key, deliberately" comment;
`entries_burler.go`'s doc line about the ungated Webster round goes.
The Burler producer's attach path already probes through `AttachGated` with the row's gate, so a resumed round carries the same spec.

A round that changed no code sees the verified tree and passes without running.
A failure re-prompts the Burler round session with the findings;
an exhausted gate is Stuck to Webster-Bouncer, as every Burler gate failure already is.
Bound: a failing verify writes no record, so a red tree Webster-Bouncer approves reaches Publish, whose verify runs because the tree differs from the record, and is Stuck there before anything is pushed (card 6).

Tests:
`entries_gate_test.go` covers the `verify` name's Env requirements and the six-name vocabulary message.
`verifygate_integration_test.go` (integration tag, real scratch repo) pins: the closure passes without running the command on a tree the record already names;
it fails with findings on a failing command, which is the verdict the shuttle gate loop re-prompts on;
it fails naming the paths on a dirty tree;
it passes on a plan with no `## verify:` section.
`testmain_test.go` calls `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, since `loomshed` gains its first integration-tagged file.

**ImpactSummary:** loom's Webster-Burler row gains a must-pass verify gate, the only row that names it; shedrecipe.Env gains one told field filled by loomcli and envkit, and every other row resolves as before.
