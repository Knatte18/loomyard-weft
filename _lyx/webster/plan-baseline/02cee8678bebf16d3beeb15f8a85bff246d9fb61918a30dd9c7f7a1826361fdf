# Card 5 — gate-resolver-darn

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/testkit/envkit#nilLegal`

**Uses:**
- `plan:internal/loomshed#NewDarnDiscussionGate`
- `internal/loomshed#NewDiscussionGate`

**Intent:** Let the existing `discussion` gate name resolve to the darn gate on a darn run, so the Darn-Write row gains the same gate entry Discussion-Write carries (D4) without a second gate name.

`Env` gains `DiscussionHistory func() ([][]byte, error)`: every committed version of the decision record, newest first, set by the darn wiring alone and nil on a loom run.
In `resolvePlainGateClosure`'s `"discussion"` case, after the two existing `requireAbsRoot` checks, a non-nil `env.DiscussionHistory` requires `env.CommitDiscussion` through `requireSeam` and returns `loomshed.NewDarnDiscussionGate(env.DecisionRecordPath, env.SupportLogPath, env.DiscussionHistory, env.CommitDiscussion)`; a nil one keeps returning `loomshed.NewDiscussionGate`.
The missing `CommitDiscussion` refusal is a fail-loud wiring guard: the darn wiring fills both seams together, so no recipe input reaches it.
`resolveGateSpec`'s doc line for `"discussion"` states the two forms and that the history seam selects the darn one.
`Env`'s field doc says the gate commits the discussion directory through `CommitDiscussion` when it passes.

`nilLegal` in `internal/testkit/envkit` gains `DiscussionHistory`, nil outside a darn run, so `FullEnv` keeps the loom form for the loom recipe tests and `TestFullEnv_FillsEverySeam` stays green.

`TestResolveGateSpec_ValidOneEntryPerVocabularyName` gains a subtest that sets `DiscussionHistory` and a counting `CommitDiscussion` on its env, writes a valid decision record and support log at the env's paths, and runs the resolved `discussion` closure twice: with a history version holding an `Added after Discussion` entry the record lacks it fails naming the heading and commits nothing, and with an empty history it passes and commits once.

**ImpactSummary:** one optional Env seam; a loom build resolves the same discussion gate as before, and only a build told the history seam gets the darn gate.

**Commit:** `5: the discussion gate entry resolves to the darn gate when told the record's history`
