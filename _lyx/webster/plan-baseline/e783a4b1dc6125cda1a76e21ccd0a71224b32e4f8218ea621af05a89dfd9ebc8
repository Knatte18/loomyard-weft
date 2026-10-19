# Card 7 — reedcli-attach-floor

**Edit:**
- `internal/reedcli#reedCLI.attachCmd`
- `docs/overview.md`

**Uses:**
- `internal/reedengine#CheckTmuxFloor`
- `internal/reedengine#Engine.TmuxPath`

**Intent:** D4: `lyx reed attach` spawns tmux itself through `handOver`, outside the `TmuxCmd` runner, so the attach command's `RunE` calls `reedengine.CheckTmuxFloor(c.eng.TmuxPath())` at its top, after the `clihelp.ShouldAbort` check and before `EnsureSession`.
Placed there, a below-floor tmux is refused first with the floor error itself, unwrapped, on the command's normal error envelope, before `EnsureSession` or `Status` run tmux and before stdio is handed to tmux, so the operator reads the tool, the version found, the floor and the way forward instead of a tmux failure.
On Windows the check admits without probing, as card 6 makes it.
The same commit rewrites the three `attach.go` texts that name `EnsureSession` and `Status` as the only fallible pre-flight steps, so each names the floor check as the first pre-flight step that can abort with an envelope error, ahead of `EnsureSession` and `Status`:
the file header comment's closing sentence ("c.eng.EnsureSession() and c.eng.Status() remain the only two pre-flight steps that can abort with an envelope error"),
the operator-facing `Long` help's list of fallible steps ("booting the session if it is cold, then checking that the server/session is up"),
and the `RunE` comment on the load-bearing pre-flight order, which gains the floor check ahead of `EnsureSession` and `Status`.
The file header's earlier claim that reading the operator's terminal size is "this file's only remaining pre-flight addition" also names the floor check, as the one such addition that can abort.
`docs/overview.md`'s reed module row, which says attach pre-flights "through booting the session if cold and checking server/session liveness", gains the tmux floor check as the first of those steps.
Card 2 also edits that file, but runs first and changes only the module tree.
No test injects a version line here: `reedcli`'s tests cannot reach reedengine's process cache, so card 6's `TestTmuxFloor` pins the check and this call is held by review, as the decision record's acceptance criterion says.

**ImpactSummary:** One added guard call at the top of the attach command, ahead of its session boot, status pre-flight and handover; the attach is unchanged on a tmux at or above 3.4.
