# Card 6 — verify-gate-told-command

**Create:**
- `plan:internal/loomcli#verifySource` -> `type verifySource struct`
- `plan:internal/loomcli#planVerifySource` -> `func planVerifySource(l *lyxcwd.Location) verifySource`

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), slots *gateslot.Pool) shuttleengine.Gate`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe/entries_gate_test.go`
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.verdict`
- `internal/landingshed/doc.go`
- `internal/landingshed/verifygate_test.go`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource) landingshed.Deps`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli/landingdeps_test.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/loomcli/landingdeps_markdone_test.go`
- `internal/loomcli/wiring_publish_integration_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/webstercli/verify_parity_integration_test.go`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `contracts/specs/shed-recipe-spec.md`

**Intent:** D6: the verify command is told to the verify gate and the landing producers by the recipe's wiring, and the gate gains the whole-diff form the darn row needs.
`NewVerifyGate` takes `command`, the told verify-command source read at each arrival, and `mergeBase`, nil or a reader of the task branch's merge base with the parent branch;
it no longer parses the plan.
With `mergeBase` nil the gate keeps the round form: an empty command passes with a warning, the lint and the impacted-set command derive from the command's latest recorded pass, and the Publish failure confirmation runs the impacted set under `tmux`.
With `mergeBase` set the gate takes the whole-diff form: a command read error is returned with its message kept, the comment lint runs from the merge base to HEAD, the told command runs in full with `Site.BaseCommand` equal to it so Publish skips on the same tree and command, and while a checked Publish failure record is present the record's failing tests and, for a `publish_verify` failure, `./...` under `tmux` join the command as the round form does on a fallback.
`Env` gains `VerifyCommand func() (string, error)`, required by the `verify` gate name, and `VerifyMergeBase func() (string, error)`, optional, which `resolvePlainGateClosure` hands to the constructor.
`landingshed.Deps` gains `VerifyFailedWayForward string`, the whole way-forward clause `verdict` appends to the recorded failed-verify reason in place of the literal Webster-Burler clause, which `verifyGate` carries from `newVerifyGate`.
`verifySource` pairs the command reader and the way-forward clause one recipe tells;
`planVerifySource` returns loom's: the plan reader `landingDeps` built inline until now, and the clause `run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests`.
`landingDeps` takes the source and fills both `Deps` fields from it; `loomPreRun` and `buildLoomShed` pass `planVerifySource`, and `wire` fills `Env.VerifyCommand` from the same source and leaves `VerifyMergeBase` nil, listed in `intentionallyNil` as the darn-only seam.
`FullEnv` fills `VerifyCommand`; `nilLegal` admits `VerifyMergeBase`.
Tests: the loomshed verify gate scenarios drive both forms, the whole-diff one asserting the lint base, the full command and the recorded pass under the told command;
the shedrecipe gate test asserts the `verify` name requires `VerifyCommand` and accepts a nil `VerifyMergeBase`;
the landing gate test asserts the recorded failure's reason ends with the told clause;
the loomcli tests assert the plan source reads the plan at call time and fills both landing fields.
The recipe spec's gate vocabulary gains `verify`, and the loomshed and landingshed docs describe the two forms and the told clause.

**ImpactSummary:** Re-signs `loomshed.NewVerifyGate` and `loomcli.landingDeps` with every caller on this card; two new `shedrecipe.Env` seams and one `landingshed.Deps` field; loom's observable behavior is unchanged.
