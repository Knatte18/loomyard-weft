# Card 4 — Every `lyx loom start` entry re-arms follow

**Create:**
- `internal/loomcli#readFollow`
- `internal/loomcli#setFollow`

**Edit:**
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`
- `internal/loomcli#TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/reedengine#Engine.FollowRun`
- `internal/loomcli#loomCLI.ensureStatusStrand`

**Intent:**
D2's loom half.
In `startCmd`, right after `c.ensureStatusStrand()` succeeds and before the watchdog spawn, still under the bootstrap lock, call `c.reed.FollowRun()` once; an error is logged at Warn and never fails the start, since the follow state is the viewer's and not the run's.
So every `lyx loom start` entry, a fresh run, a re-entrant start on a live run and a resume of a parked run, turns follow on and selects the relevant window, whichever of keep, replace or add `ensureStatusStrand` took.
`ensureStatusStrand` itself stays as it is, so no strand add, replace or resume turns follow on by itself, and `lyx loom resume`, which never brings reed up, leaves the follow state as it is.
The bound: a start may move a viewer who turned follow off, which D2 accepts; no other loom verb calls `FollowRun`.

In `smoke_drivingsurface_test.go`, `readFollow` and `setFollow` read and set the session option `@lyx_follow` through the tmux binary `tmuxBinaryPath` returns, on `-L <eng.Socket()>` and the exact target `=<eng.SessionName()>`; `readFollow` returns the trimmed `display-message -p -t =<session>: '#{@lyx_follow}'` value, the same call shape `smoke_starttail_test.go` already makes.
Both are kept as named helpers because the rows below each read and set the option several times.
`TestSmokeStatusStrandAcrossDriverSeeds` gains:
- in its llm-driver subtree, `start leaves exactly one status strand` sets follow off after adding the pre-existing status strand, so the start keeps or replaces it as a second run in a reused pair does, and asserts after the start that `readFollow` returns `on` and that the session's current window is the window holding the status strand's pane, the overview, which is the relevant window since the providerless driver launch leaves no live agent;
- in its go-driver subtree, after the step subtests, which leave the go-seeded run halted with no driver, a subtest `loom resume leaves a follow-off session off`: with reed up, follow set off and the current window read, `lyx loom resume` exits 0 with the envelope message `a Go driver was spawned`, which proves it took the wake path rather than an early refusal, and `readFollow` still returns `off` and the current window is the one read before.
`TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` sets follow off before its second bootstrap, a start entering a run whose driver is live, the same start path a parked run's resume through `lyx loom start` takes up to the `FollowRun` call, and asserts after it that `readFollow` returns `on` and that the current window is the live driver strand's window, the newest live agent.
Each file's header comment adds that a start re-arms follow and a resume does not.

`doc.go`'s start paragraph adds that the start turns follow on and selects the run's relevant window after ensuring the status strand, and that `lyx loom resume` does not.

**ImpactSummary:** One call added in `startCmd` behind the status strand, Warn-only on failure; one existing smoke test gains rows and two helpers; no other loom verb or test changes.

**Commit:** 4: loom: every start entry turns the session's follow on
