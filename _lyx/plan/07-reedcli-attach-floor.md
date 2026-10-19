# Card 7 — reedcli-attach-floor

**Edit:**
- `internal/reedcli#reedCLI.attachCmd`

**Uses:**
- `plan:internal/reedengine#CheckTmuxFloor`
- `internal/reedengine#Engine.TmuxPath`

**Intent:** D4: `lyx reed attach` spawns tmux itself through `handOver`, outside the `TmuxCmd` runner, and its pre-flight degrades a failed tmux call to the bare argv, so the attach command's `RunE` calls `reedengine.CheckTmuxFloor(c.eng.TmuxPath())` before `AttachArgv` and `handOver`.
A refusal returns the error through the command's normal error envelope, before stdio is handed to tmux, so the operator reads the tool, the version found, the floor and the way forward instead of a tmux failure.
On Windows the check admits without probing, as card 6 makes it.
No test injects a version line here: `reedcli`'s tests cannot reach reedengine's process cache, so card 6's `TestTmuxFloor` pins the check and this call is held by review, as the decision record's acceptance criterion says.

**ImpactSummary:** One added guard call ahead of the attach handover; the attach is unchanged on a tmux at or above 3.4.
