# Card 11 — arbiter-cmd-wiring

**Create:**
- `plan:internal/loomcli#runnerArbiterShuttle` -> `type runnerArbiterShuttle struct`
- `plan:internal/loomcli#loomCLI.arbiterCmd` -> `func (c *loomCLI) arbiterCmd() *cobra.Command`
- `plan:internal/loomcli#loomCLI.arbiterDepsFor` -> `func (c *loomCLI) arbiterDepsFor() (arbiterDeps, error)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestStatusCommitPathspec`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/loomcli#arbiterVerb`
- `internal/loomengine#ResolveArbiter`
- `internal/shedrun#ArbiterDir`
- `internal/shedrun#ArbiterRel`
- `internal/shedrun#ArbiterAttemptDir`
- `internal/shedrun#ArbiterMarker`

**Intent:**
Mount the verb as `lyx loom arbiter <run-id> --envelope <path> [--report <path>]` and wire its seams, in `arbiter.go` beside the body.

`arbiterCmd` builds the cobra command (`PATTERN-cli-cobra`): `Use` `arbiter <run-id>`, `Args` exactly one, `Short` and `Long` saying only the run's driver runs it, at a stop it cannot settle, and what the envelope carries, the `clihelp.AudienceRole` annotation, which keeps it out of the orch's command index (D1), a required `--envelope` and an optional `--report`; its `RunE` builds the deps and runs `arbiterVerb`, reporting a deps error as a JSON error envelope.
The loom pre-run arms it with the full `wire` like any non-lightweight verb, resolving the run id from its positional argument, so `c.cfg`, `c.registry`, `c.runner`, `c.routing` and `c.location` are set for loom and darn runs alike.

`arbiterDepsFor` resolves `loomengine.ResolveArbiter(c.cfg, c.registry)`, takes the status paths from `c.shedPaths`, the arbiter directory, attempt directories and marker from `internal/shedrun`, the producers from `c.routing`, the Bouncer subdirectory through `loomrecipe.BouncerRunSubdir` under `loomengine.LoomReviewsDir`, the pending circling decision through `shedadapters.PendingCirclingDecision`, the Publish failure target through `verifytree.ReadPublishFailure` at the Publish row (`loomshed.NamePublish`), the interrupt policy through `loomshed.InterruptPolicyFor`, the decision record and support log through `loomengine.DiscussionDecisionRecord` and `DiscussionSupportLog`, and `time.Now`.
`runnerArbiterShuttle` adapts `c.runner` to `arbiterShuttle`: `Start` returns the `*shuttleengine.Run`, and `AttachIfLive` and `StopStrand` delegate.
`Command` adds `c.arbiterCmd()` to the loom subtree.

`statusCommitPathspec` stages `shedrun.ArbiterRel(location, runID)` when `shedrun.ArbiterDir` holds a file, as it does for the drive reports, so each attempt's prompt, verdict and envelope are committed with the run records (D9).
Add a row to `TestStatusCommitPathspec` for an arbiter directory holding an attempt file, and assert an empty one is left out.

Describe the verb in `doc.go`: when the driver runs it, the attempt records and marker, the stop key, the budget, the re-run behavior, and that a pair's session end ends a live arbiter strand with the pair's other strands (`PATTERN-pair-teardown`).

**ImpactSummary:** one new loom verb and one more pathspec entry; existing verbs and their wiring are unchanged.
