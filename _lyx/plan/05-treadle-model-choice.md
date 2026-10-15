# Card 5 — Treadle's profile carries judge and attempt ModelChoices

**Edit:**
- `internal/treadleengine#Profile`
- `internal/treadleengine#AttemptInput`
- `internal/treadleengine#judgeInputs`
- `internal/treadleengine#runJudgeCall` -> `func runJudgeCall(sh Shuttle, name string, template []byte, values map[string]string, framing judgeFraming, round int, choice shuttleengine.ModelChoice, fallback JudgeVerdict, label string) (JudgeVerdict, string, bool)`
- `internal/treadleengine#runCircling`
- `internal/treadleengine#runMilestone`
- `internal/treadleengine#runTargeting` -> `func runTargeting(stencilsDir, parentName string, sh Shuttle, name string, round int, previousHandoffPath, seedPath string, choice shuttleengine.ModelChoice) (string, bool)`
- `internal/treadleengine#Engine.Run`
- `internal/treadleengine#Engine.runPreRoundTargeting`
- `internal/treadleengine#Engine.runRound`
- `internal/treadleengine/doc.go`
- `internal/treadleengine#TestEngine_AttemptInputPopulation`
- `internal/treadleengine#TestRunCircling`
- `internal/treadleengine#TestRunMilestone`
- `internal/treadleengine#TestSpawnedRoles_SkillAndParentDirective`
- `internal/treadleengine#TestRunTargeting_FailSafe`
- `internal/treadleengine_test#TestSmokeJudgeCirclingToyFixture`

**Intent:**
D3 for treadle.
`Profile` replaces `JudgeModel`, `JudgeEffort`, `Model` and `Effort` with `Judge` and `Attempt`, each a `shuttleengine.ModelChoice`.
`AttemptInput` and `judgeInputs` replace their `Model` and `Effort` with one `Choice shuttleengine.ModelChoice`, so the engine reaches the attempt and the judges; `runJudgeCall` and `runTargeting` take the choice and set `ModelChoice: choice` on their spec.
`Engine.Run` hands `p.Judge` to the circling and milestone judges, `runPreRoundTargeting` hands it to `runTargeting`, and `runRound` puts `p.Attempt` on each `AttemptInput`.
Treadle declares nothing (D5): no field here adds a requirement.
`doc.go` names the two choices.

Tests:
`TestEngine_AttemptInputPopulation` sets `Attempt` with `Engine: "b"` and asserts every attempt's `Choice` equals it;
`TestRunCircling` and `TestRunMilestone` set the judge input's choice with `Engine: "b"` and assert the spawned spec's `Engine`, model and effort.
`TestSpawnedRoles_SkillAndParentDirective` also asserts that the circling, milestone and targeting specs it captures carry no `Requires`;
it, `TestRunTargeting_FailSafe` and `TestSmokeJudgeCirclingToyFixture` move to the choice argument and field.

**ImpactSummary:** treadle-internal only (no production caller outside the package): the profile, two input types, two spawners and their tests change.

**Commit:** 5: treadle's profile carries judge and attempt ModelChoices
