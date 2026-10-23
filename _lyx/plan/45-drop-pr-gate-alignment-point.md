# Card 45 — drop-pr-gate-alignment-point

**Delete:**
- `internal/alignment#PointPRGate`
- `internal/alignment#MeasureDarn`
- `contracts/stencils/loom/loom-fragment-pr-gate-darn.md`
- `contracts/stencils#LoomFragmentPRGateDarn`

**Edit:**
- `internal/alignment#Attempt`
- `internal/alignment#SpawnInput`
- `internal/alignment#PromptInput`
- `internal/alignment#Prompt`
- `internal/alignment#Session.Begin`
- `internal/alignment#TestAttempts`
- `internal/alignment#TestPrompt`
- `contracts/stencils/loom/loom-template-alignment.md`
- `contracts/stencils#entries`
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#templateConfig`
- `internal/loomengine/template.yaml`
- `plan:internal/loomengine#retiredLoomKeys`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestMigrateConfig_RetiredKeys`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/reportengine#runAttempts`
- `internal/reportengine#settlerOf`

**Extends:**
- `internal/alignment/doc.go`

**Intent:** The alignment agent's `pr-gate` point and `alignment_pr_approve` go (D13); the parent-review alignment point before the Discussion perch stays, with `skip_alignment` and `alignment_timeout_min` (Scope, Out).
`alignment` drops `PointPRGate`, the pull-request fields of `Attempt`, `SpawnInput` and `PromptInput` that only that point set, and the pr-gate branch of `Prompt` and `Session.Begin`; the alignment stencil drops its pr-gate section and the darn measure fragment goes, while the loom measure fragment the parent-review aligner names stays.
`loom.yaml` loses `alignment_pr_approve`: `Config` drops `AlignmentPRApprove`, the template drops the key, `retiredLoomKeys` gains it so the migration drops it everywhere, and `skip_alignment`'s description names parent review alone.
The convergence and timing report stop counting PR-Gate alignment attempts and stop crediting a PR-Gate stop to an alignment (`runAttempts`, `settlerOf`), so the report's alignment-verdict figures for that point go (D13); an old run's pr-gate attempt record reads as an unknown point and is skipped, which a `TestAttempts` row pins.
`doc.go` names one point.

**ImpactSummary:** One alignment point, one fragment and one loom.yaml key go; their readers are the alignment package, the report's attempt figures and two batten fixtures, all listed.
