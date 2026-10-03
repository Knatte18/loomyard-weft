# Card 8 — Webster-Burler carries a `verify` gate

**Edit:**
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_gate_test.go`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomcli/wiring.go`
- `internal/testkit/envkit/envkit.go`

**Create:**
- `internal/shedrecipe/testmain_test.go`
- `internal/shedrecipe/entries_gate_verify_integration_test.go`

**Uses:**
- `internal/verifytree/verify.go`

**Intent:**
D8: a Burler round that changed code ends with a passing verify.

`shedrecipe.Env` gains `VerifyDir`, filled by `loomcli` wiring with `verifytree.Dir(anchorPath)` and by `envkit` with an absolute path under its fixture dir.
`resolveGateSpec`'s closed vocabulary gains `verify`:
it requires absolute `Env.AnchorPath`, `Env.WorktreeRoot` and `Env.VerifyDir`, and returns a must-pass closure that, at each arrival, parses the plan under `Env.AnchorPath` for its `## verify:` command and calls `verifytree.Verify` in `Env.WorktreeRoot` with site label `<row name> gate` (`Webster-Burler gate` on the shipped row) and the closure's attempt count.
`StatusPassed` and `StatusSkipped` pass;
`StatusDirty` fails with the dirty paths;
`StatusFailed` fails with the exit code, the log path and the log's tail;
a plan with no `## verify:` section passes with a logged warning;
a plan read error or a cancelled verify is a returned error.
The vocabulary error message names all six values.

`loom-recipe.yaml`'s Webster-Burler row gains `gates: [{name: verify, attempts: 3}]`, replacing its "No gates key, deliberately" comment;
`entries_burler.go`'s doc line about the ungated Webster round goes.
The Burler producer's attach path already probes through `AttachGated` with the row's gate, so a resumed round carries the same spec.

A round that changed no code sees the verified tree and passes without running.
A failure re-prompts the Burler round session with the findings;
an exhausted gate is Stuck to Webster-Bouncer, as every Burler gate failure already is.
Bound: a failing verify writes no record, so a red tree Webster-Bouncer approves reaches Publish, whose verify runs because the tree differs from the record, and is Stuck there before anything is pushed (card 6).

Tests:
`entries_gate_test.go` covers the `verify` name's Env requirements and the six-name vocabulary message.
`entries_gate_verify_integration_test.go` (integration tag, real scratch repo) pins: the closure passes without running the command on a tree the record already names;
it fails with findings on a failing command, which is the verdict the shuttle gate loop re-prompts on.
`testmain_test.go` calls `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, since the package gains its first integration-tagged file.

**ImpactSummary:** loom's Webster-Burler row gains a must-pass verify gate; shedrecipe.Env gains one told field filled by loomcli and envkit.
