# Card 2 — Shuttle stamps a launch's CreelHooks onto its strand

**Edit:**
- `internal/shuttleengine#Launch`
- `internal/shuttleengine#Runner.launch`
- `internal/shuttleengine#TestRunner_Start_HappyPath`

**Uses:**
- `internal/reedengine#AddSpec`
- `internal/shuttleengine#fakeEngine`

**Intent:**
D5 has shuttle mark a strand `CreelHooks` on each launch whose rendered settings carry creel's PostToolUse, Stop and waiter hooks.
Only the engine knows what its settings render, so add `CreelHooks bool` to `Launch`, documented as: the launch's settings carry creel's hook delivery (the tool-boundary claim, the turn-end Stop delivery and the idle waiter), so creel may ring the strand's bell.
`Runner.launch` passes `launch.CreelHooks` on the `reedengine.AddSpec` it builds, beside `Provider`.
`Runner.Open` builds no such launch and its `AddSpec` keeps the zero value, so a `lym shuttle open` session is never marked, as D5 requires.
`TestRunner_Start_HappyPath` scripts its fake engine's prepared launch with `CreelHooks: true` and adds `CreelHooks: true` to the `AddSpec` its "wires AddSpec verbatim" subtest wants, so the field reaches reed only by this wiring.
Claude's `Prepare` sets the field in card 14, when it renders those hooks.

**ImpactSummary:** one Launch field and one AddSpec field set in Runner.launch; every other Launch producer, fakes and Codex-less engines included, leaves it false, so no other caller changes.
