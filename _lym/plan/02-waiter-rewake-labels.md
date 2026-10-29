# Card 2 — waiter-rewake-labels

**Edit:**
- `internal/shuttleengine/claudeengine#hookCommand`
- `internal/shuttleengine/claudeengine#buildSettings`
- `internal/shuttleengine/claudeengine#TestBuildSettings_CreelMailHook`
- `internal/shuttleengine/claudeengine/doc.go`

**Create:**
- `plan:internal/shuttleengine/claudeengine#creelRewakeMessage` -> `const creelRewakeMessage = "[creel] Mail that arrived while you were idle:"`
- `plan:internal/shuttleengine/claudeengine#creelRewakeSummary` -> `const creelRewakeSummary = "creel mail"`

**Intent:**
Decision D4: the idle waiter's Stop hook entry carries creel's labels for the wake, so the model reads `[creel] Mail that arrived while you were idle: <lines>` instead of `Stop hook blocking error from command "<hook name>":`, and the operator's notification row reads `creel mail`.

The struct `hookCommand` gains two fields, `RewakeMessage string` tagged `json:"rewakeMessage,omitempty"` and `RewakeSummary string` tagged `json:"rewakeSummary,omitempty"`, so every other hook entry renders as today.
`buildSettings` sets them to the two new constants on the waiter entry (`creelWaitHookCommand`, `AsyncRewake`, `creelWaitTimeout`) only.
The constants' doc comments say both keys are in Claude Code 2.1.312's hook schema marked `@internal`, that the summary is fixed for the session because Claude Code takes a per-wake summary only from a hook of an Anthropic-marketplace plugin, and that the message opens with `[creel]` so the wake's first line matches the creel trust sentence.
`internal/shuttleengine/claudeengine/doc.go`'s waiter sentence adds the two keys, that the terminal draws the wake as one `creel mail` row and never its body (so D6's echo gives the operator the mail), and the Bound that a session launched before the change keeps the default labels until relaunched.

Test (tier0): `TestBuildSettings_CreelMailHook`'s `stopHook` gains `rewakeMessage` and `rewakeSummary`, read from each Stop hook; the InAHub row wants both on the waiter hook and neither on the merged turn-end hook, and the Standalone row wants neither.

**ImpactSummary:** Two omitempty fields on one hook struct, set on one entry; every other rendered hook entry is byte-identical, and one test row pins it.
