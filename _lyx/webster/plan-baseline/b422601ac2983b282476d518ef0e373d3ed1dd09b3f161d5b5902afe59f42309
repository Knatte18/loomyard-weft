# Card 12 — runner-pause-wiring

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#battenCLI.wire`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/shuttlecli#Command`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/webstercli#TestWire_ModeHubSelectsHubMode`
- `internal/webstercli#TestWire_StandaloneMode`

**Intent:**
Tell every shuttle runner that starts agent work its pause geometry (card 8), so every agent-session spawn path waits while a pause covers its pair (D4).
Each runner is told `hubgeom.PauseGeometry` of the location it is built for, right after its constructor:
- loom's wiring (`loomCLI.wire`), which every loom producer, advisor, Burler round and the driver's Master start through;
- batten's wiring (`battenCLI.wire`), for the runner it builds over the task worktree;
- burler's and webster's hub wiring (`wireHub`), and their standalone wiring (`wireStandalone`) with `standalonegeom.PauseGeometry(res.StateDir)`, for a standalone run that a hub-scope entry in its own state directory would cover;
- shuttle's own CLI (`Command`'s pre-run), whose `run` verb starts agents.
A `PauseGeometry` error, which only an unreadable `.git` entry gives, refuses the wiring as `ReedGeometry` does for the same location.
The orch's runner is not told one: `lyx orch start` is the operator's own session and not held (D4).

Tests: `TestWire_DefaultConfig` asserts `c.runner.PauseGeometry()` equals `hubgeom.PauseGeometry(location)`'s answer; `TestWire_ModeHubSelectsHubMode` asserts the same for webster's hub runner, and `TestWire_StandaloneMode` asserts the standalone runner's record path lies under the derived state directory with an empty slug.
The held start itself is pinned at the runner by card 8's `TestRunner_Start_WaitsWhileAPauseCoversItsPair`.

**ImpactSummary:** One told value after each runner construction in seven wiring sites (six wire functions and shuttle's `Command` pre-run); no signature changes and no new behaviour beyond card 8's held start.
