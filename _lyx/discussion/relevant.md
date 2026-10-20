# Relevant code

- `contracts/recipes/darn-recipe.yaml` — darn's rows and the two verify gates on Darn-Write and Darn-Burler-Fix
- `internal/loomshed/verifygate.go` `NewVerifyGate` — the verify gate's round and whole-diff forms, which darn's gates use today in the whole-diff form
- `internal/loomcli/darnwiring.go` `wireDarn`, `darnVerifySource`, `pairMergeBase` — sets the darn verify source, merge base and whole-diff form
- `internal/loomcli/landingdeps.go` — tells landingshed the verify command source and the fabric seams for changed paths and merge base
- `internal/impactset/doc.go` `Derive`, `FindGuardTests` — the impacted-set command and the guard-marker scan
- `internal/impactset/derive.go` `buildCommand` — assembles the derived command's raw build, vet and test steps
- `internal/landingshed/doc.go` — Merge-In, Publish and Finalize verify behaviour, `publish_verify` and the failure record
- `internal/verifytree/doc.go` — verified-tree record keyed per command, base command kept
- `internal/darnengine/doc.go` `LoadConfig`, `VerifyCommand` — darn.yaml's keys and loading
- `internal/gatecli/doc.go` — `lyx gate test` flags and package rules
- `internal/shedrecipe/entries_gate.go` — builds the verify gate from Env fields
- `contracts/stencils/darn/darn-template-write.md` — tells the writer the handoff runs the full verify
- `pattern/PATTERN-verified-tree.md` — the list of verify sites and their base commands
- `pattern/PATTERN-test-speed.md` — the `//lyx:guard` rule and its bound
