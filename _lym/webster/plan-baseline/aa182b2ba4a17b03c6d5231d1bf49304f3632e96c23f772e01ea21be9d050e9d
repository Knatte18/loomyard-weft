# Card 5 — strand-free-verify-commands

**Edit:**
- `internal/battencli#runDeploy`
- `internal/battencli#runRowCommand`
- `internal/cardverify#Runner.runLine`
- `internal/cardverify#TestRunner`
- `//internal/cardverify/doc.go`

**Create:**
- `plan:internal/battencli#commandEnv` -> `func commandEnv(environ []string) []string`
- `plan:internal/battencli#TestCommandRunners_DropTheStrand` -> `func TestCommandRunners_DropTheStrand(t *testing.T)`

**Uses:**
- `internal/agentname#StrandNameEnv`
- `internal/battencli#TestRunDeploy_WritesTheLogAndReportsExitCodeAndTimeout`

**Intent:** The Constraints' strand removal: every `verifyrun.Run` caller runs its command without `agentname.StrandNameEnv`, as `verifytree` already does, so a gate call inside an operator-written command never takes D2's capped form and never detaches a Go caller.

`commandEnv(environ)` returns `environ` without every entry that starts with `LYM_STRAND_NAME=`.
`runDeploy` and `runRowCommand` pass `commandEnv(os.Environ())` to `verifyrun.Run` in place of the nil environment they pass today, so the deploy command and a batten Command row's command still inherit the rest of the environment.
`cardverify.Runner.runLine` removes the strand from the environment it builds (`gateslot.StripPrebuilt(os.Environ())` less the strand entry) for every line, a gate call and an unslotted line included, so a nested gate call in a line runs in-process inside and outside a hub (D2).
The cardverify doc says no line sees the strand.

Tests:
- `TestCommandRunners_DropTheStrand`, in `deploy_integration_test.go` beside the deploy test (build line `//go:build tier1 && !windows`), not parallel since it sets the process environment: with `t.Setenv(agentname.StrandNameEnv, "sh:slug:implementer")` and `t.Setenv("LYM_STRAND_KEPT", "kept")`, both `runDeploy` and `runRowCommand` run `test -z "$LYM_STRAND_NAME" && test "$LYM_STRAND_KEPT" = kept` and report exit code 0, so each runner drops the strand and keeps the rest of the environment; a runner still passing a nil environment fails its row with exit 1.
  `commandEnv` is reached through those rows and gets no test of its own.
- `TestRunner`, tier1, gains a subtest among its leading non-parallel ones: with `t.Setenv(agentname.StrandNameEnv, "sh:slug:implementer")`, the line `test -z "$LYM_STRAND_NAME"` passes through `RunLine`, unslotted and slot-held alike.

**ImpactSummary:** three command runners lose one environment variable; no caller's signature changes and the batten rows still get the operator's environment.
