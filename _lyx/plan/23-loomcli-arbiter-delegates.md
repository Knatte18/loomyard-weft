# Card 23 — loomcli-arbiter-delegates

**Edit:**
- `internal/loomcli#arbiterVerb` -> `func arbiterVerb(out io.Writer, deps shedarbiter.Deps, in shedarbiter.Input) int`
- `internal/loomcli#loomCLI.arbiterCmd`
- `internal/loomcli#loomCLI.arbiterDepsFor` -> `func (c *loomCLI) arbiterDepsFor() (shedarbiter.Deps, error)`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#loomCLI.loomConsumeAttempt`
- `internal/loomcli#arbiterEnvelopeFields`
- `internal/loomcli/doc.go`

**Delete:**
- `internal/loomcli#arbiterInput`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterStopEnvelope`
- `internal/loomcli#arbiterRefusal`
- `internal/loomcli#arbiterCall`
- `internal/loomcli#newArbiterCall`
- `internal/loomcli#arbiterCall.settle`
- `internal/loomcli#arbiterCall.settleExisting`
- `internal/loomcli#arbiterCall.attachAndConclude`
- `internal/loomcli#arbiterCall.settleFresh`
- `internal/loomcli#arbiterCall.recordStartedArbiter`
- `internal/loomcli#arbiterCall.checkEligible`
- `internal/loomcli#arbiterCall.concludeWait`
- `internal/loomcli#arbiterCall.stopAndHandOn`
- `internal/loomcli#arbiterCall.stopRunAndHandOn`
- `internal/loomcli#arbiterCall.conclude`
- `internal/loomcli#arbiterCall.resumePrecondition`
- `internal/loomcli#arbiterCall.handOn`
- `internal/loomcli#arbiterCall.finish`
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomcli#arbiterStencilName`
- `internal/loomcli#arbiterVerdictGateAttempts`
- `internal/loomcli#arbiterSkills`
- `internal/loomcli#arbiterPromptInput`
- `internal/loomcli#arbiterPrompt`
- `internal/loomcli#arbiterSpec`
- `internal/loomcli#pathOrNone`
- `internal/loomcli#arbiterVerdictGate`
- `internal/loomcli#arbiterFake`
- `internal/loomcli#arbiterFake.Start`
- `internal/loomcli#arbiterFake.AttachIfLive`
- `internal/loomcli#arbiterFake.StopStrand`
- `internal/loomcli#arbiterFakeRun`
- `internal/loomcli#arbiterFakeRun.Wait`
- `internal/loomcli#arbiterFakeRun.Stop`
- `internal/loomcli#arbiterFakeRun.StrandGUID`
- `internal/loomcli#arbiterFixture`
- `internal/loomcli#arbiterTestRun`
- `internal/loomcli#newArbiterFixture`
- `internal/loomcli#arbiterFixture.haltAtVerifyRow`
- `internal/loomcli#arbiterFixture.arbiterDir`
- `internal/loomcli#arbiterFixture.markerPath`
- `internal/loomcli#arbiterFixture.attemptDir`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli#arbiterFixture.run`
- `internal/loomcli#arbiterFixture.priorAttempt`
- `internal/loomcli#writeArbiterVerdictFile`
- `internal/loomcli#arbiterResumeVerdict`
- `internal/loomcli#arbiterHandOnVerdict`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli#TestArbiterVerb_Rerun`
- `internal/loomcli#TestArbiterVerb_VerdictGate`
- `internal/loomcli#TestArbiterVerb_VerdictOutcomes`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `internal/loomcli#TestArbiterPrompt`
- `internal/loomcli#TestArbiterSpec`

**Uses:**
- `plan:internal/shedarbiter#Deps`
- `plan:internal/shedarbiter#Input`
- `plan:internal/shedarbiter#NewCall`
- `plan:internal/shedarbiter#Call.Settle`
- `plan:internal/shedarbiter#ConsumeAttempt`
- `plan:internal/shedarbiter#Seam`
- `plan:internal/loomcli#arbiterRecipeDescription`

**Intent:**
Point loom and darn at `internal/shedarbiter` and remove `loomcli`'s own arbiter call, prompt and consumption (D8).
`arbiterDepsFor` returns `shedarbiter.Deps`, its prompt input carrying `arbiterRecipeDescription(c.recipe)`; `arbiterVerb` runs `shedarbiter.NewCall` and `Settle` and prints `arbiterEnvelopeFields` of the envelope as today, so `lyx loom arbiter`, the LLM driver's command, keeps its output; `arbiterCmd` is unchanged in flags and help.
`goDriverDepsFor`'s `Arbiter` seam is `shedarbiter.Seam(deps, runID, ...)` over `arbiterstop.EscalationEligible`, and `loomConsumeAttempt` calls `shedarbiter.ConsumeAttempt`.
The removed members' tests were carried over to `shedarbiter` on the previous card; `arbiterHandBackWayForward`, `arbiterAbsentPath` and `arbiterAmendmentTarget` stay, since the deps and the amendment target read them, and `pathOrNone`, read only by the deleted prompt, goes.
`loomcli/doc.go`'s arbiter paragraph says the verb's body is `shedarbiter`'s.

**ImpactSummary:** loomcli's arbiter verb, driver seam and consume hook delegate to shedarbiter; the deleted members and tests have no caller left outside this card.
