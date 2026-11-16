# Card 25 — batten-fixer-timeout

**Edit:**
- `internal/battencli#fixerTimeout`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`

**Uses:**
- `internal/weaverengine#ActiveOrSetting`
- `internal/weaverengine#LoadConfig`
- `internal/weaverengine#SaveState`

**Intent:** Batten's watch bounds a Weaver run's verify-fixer strand by `weaver.yaml`'s `fixer_timeout_min` (D10).

- `fixerTimeout` for a `verify-fixer` role asks the seam's advisory form for the task's active implementer: Weaver reads `weaverengine.LoadConfig`'s `FixerTimeoutMin`; Webster keeps `recovery_timeout_min`.
  A seam refusal falls back to the setting, as the other advisory readers do, and logs its reason.

`TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` gains rows over a task location with a begun Weaver state (planted through `weaverengine.SaveState`) and a `weaver.yaml` setting `fixer_timeout_min: 10`, below Webster's `recovery_timeout_min` of 60:
a verify fixer registered 5 minutes ago is a wait; one registered 30 minutes ago, past Weaver's bound and within Webster's, is no wait;
with a corrupt Webster `state.json` beside the Weaver state, the fallback still reads Weaver's bound when the setting names `weaver`.

**ImpactSummary:** One batten timeout read branches on the active implementer; Webster runs read the same key as before.
