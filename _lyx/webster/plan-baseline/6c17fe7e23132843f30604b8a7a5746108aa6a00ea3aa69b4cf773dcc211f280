# Card 17 — single-llm-delete

**Create:**
- `plan:internal/shedadapters#TestMultiLLMProducer_MapsTheLeadResult` -> `func TestMultiLLMProducer_MapsTheLeadResult(t *testing.T)`
- `plan:internal/loomcli#TestSmokeLeadSeat_HarvestsAFinishedRunWithReedStateGone` -> `func TestSmokeLeadSeat_HarvestsAFinishedRunWithReedStateGone(t *testing.T)`
- `plan:internal/loomcli#TestSmokeLeadSeat_RerunRemovesTheSupersededStrand` -> `func TestSmokeLeadSeat_RerunRemovesTheSupersededStrand(t *testing.T)`

**Delete:**
- `internal/shedadapters#SingleLLMProducer`
- `internal/shedadapters#SingleLLMProducer.Call`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#NewSingleLLMProducer`
- `internal/shedadapters#NewSingleLLMProducerGated`
- `internal/shedadapters#SpecSource`
- `internal/shedadapters#singleLLMEngineLabel`
- `//internal/shedadapters/singlellm_test.go`
- `//internal/shedrecipe/entries_singlellm.go`
- `//internal/shedrecipe/entries_singlellm_test.go`
- `internal/shedadapters#TestMultiLLMProducer_MapsTheChairResultAsSingleLLMDoes`
- `internal/loomcli#TestSmokeSingleLLM_HarvestsAFinishedRunWithReedStateGone`
- `internal/loomcli#TestSmokeSingleLLM_RerunRemovesTheSupersededStrand`

**Edit:**
- `internal/shedadapters#Shuttle`
- `//internal/shedadapters/shuttle.go`
- `internal/shedadapters#Bouncer.settle`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters/doc.go`
- `internal/shedadapters/gateattempts.go`
- `internal/shedadapters/archive.go`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/loomshed#discussionWrite`
- `internal/loomshed#discussionWrite.Call`
- `internal/loomshed#InterruptPolicies`
- `internal/shuttleengine#Runner.attach`
- `internal/mergeresolve/spec.go`
- `internal/loomcli#loomCLI.wire`
- `internal/shedbuild#engineMinimalConfig`
- `internal/shedbuild#TestBuild_Errors`
- `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`
- `internal/shuttleengine#TestAttach_RunningRecordSatisfiedFileContract_HarvestsNotRespawn`
- `internal/loomshed/planwrite_test.go`
- `internal/shedbuild/doc.go`
- `//contracts/specs/shed-recipe-spec.md`
- `//docs/overview.md`

**Intent:** D10's removal: `SingleLLMProducer`, `NewSingleLLMProducer`, `NewSingleLLMProducerGated`, `SpecSource` and the generic `SingleLLM` registry row go, now that no row runs on them (cards 11 to 15).

- The single producer's members are deleted from `internal/shedadapters/shuttle.go` (card 16's rename of `singlellm.go`), whose file header then names the `Shuttle` seam alone, and `singlellm_test.go` is deleted; `archiveStaleOutputs` (in `archive.go`) and `gateAttemptsPointer` stay, used by `MultiLLMProducer` and the Burler producers, and their file headers stop naming `SingleLLMProducer`.
- `internal/shedrecipe/entries_singlellm.go` and its test file are deleted, the `SingleLLM` row leaves `registry` (PATTERN-shed-recipe-registry), and `minimalEntries`, `TestNames` and the coverage guard's `coverageGuardAllowedUnreachableEngines` (with its comment) drop it; `shedbuild`'s `engineMinimalConfig`, which `TestBuild_EveryRegisteredEngineBuilds` walks over every registry name, drops its `SingleLLM` entry and the stencil constant only that entry used, and `TestBuild_Errors` drops its "single-LLM missing its stencil key" row, whose engine no longer exists.
- `Shuttle`'s doc comment is rewritten: the seam Bouncer drives and the Burler rows probe through, with `RunGated` and `AttachGated` beside `Run` and `Attach`; no mention of a single-LLM producer.
- Comments that cite `SingleLLMProducer` as the precedent are reworded to cite `MultiLLMProducer` or state the rule directly: `Bouncer.settle`, `Bouncer.runSeedSpawn`, `Bouncer.judgeCall`, `shedadapters/doc.go` (the adapter list, the outcome mapping, the clock, the probe-before-archive and the cancellation sections), `loomshed`'s `discussionWrite`, `discussionWrite.Call` and `InterruptPolicies`, `shuttleengine`'s `Runner.attach`, `mergeresolve/spec.go` (which cites the deleted `loomengine.DiscussionSpec`'s resolution shape), `Env`'s `Shuttle` field and `loomCLI.wire`'s comment over it (Bouncer and the Burler probes read it, no writer row does).
- The doc comments of `Env`'s `AnchorPath`, `WorktreeRoot` and `StencilsDir` fields, which name `SingleLLM` or its `anchor_path` and `output_files` tokens among their readers, drop it and name only the readers that remain.
- `shed-recipe-spec.md` and `docs/overview.md` stop listing `SingleLLMProducer` among the adapters, and the spec's `Config` bullet gives `MultiLLMProducer` in place of `SingleLLMProducer` as its example of one `Engine` serving many rows.
- Test comments that cite the single producer are reworded the same way: the doc comments of `TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact` and `shuttleengine`'s `TestAttach_RunningRecordSatisfiedFileContract_HarvestsNotRespawn`, and `loomshed`'s `planwrite_test.go` header.
- `shedbuild`'s package doc drops the single-LLM constructor from the registry constructors that reach disk at construction time, leaving the bouncer and burler-round constructors and their three effects.

Tests:
- `TestMultiLLMProducer_MapsTheLeadResult` replaces `TestMultiLLMProducer_MapsTheChairResultAsSingleLLMDoes`, keeping its table of lead results (done, done with a failed gate, died, timeout, not started with and without a start error, unrecognized) and asserting each outcome, pointer and error shape directly instead of comparing with the deleted producer.
- `TestSmokeLeadSeat_HarvestsAFinishedRunWithReedStateGone` and `TestSmokeLeadSeat_RerunRemovesTheSupersededStrand` replace the two `TestSmokeSingleLLM_*` tmux-tier tests with the same scenarios driven through `shedadapters.NewMultiLLMProducerSourced` over `seatengine.New(seatengine.RunnerShuttle(runner), geometry)` and a one-seat table whose lead declares the same output file, role prefix and round: a finished run with reed's state file gone is harvested Done, and a rerun over a halted run removes the superseded strand before starting fresh. The harvest test's stand-in run is started by `runner.Start` with the spec shape the single producer gave it, so it is the criterion's case of a run started before the deploy that the seat producer attaches to instead of respawning. In the rerun test the second stand-in run, the one with other outputs that the probe must leave alone, takes the role `other` instead of `discussion`, since the seat engine refuses a lead whose strand name another live strand holds (`checkName`), a check the single producer never made. The lead's stencil is a one-line stencil written into a `stencilkit`-seeded directory.

`SpecSource`'s last readers are gone: the `Env` fields cards 11 to 15 replaced, and `envkit.FullEnv`'s local `specOver`, which card 14 dropped with the last field it filled.

**ImpactSummary:** Removes the single producer, its registry row and its tests; every production caller left with cards 11 to 15, and the remaining references are comments listed here.
