# Card 9 — strand-placements

**Edit:**
- `internal/loomcli#statusStrandAddSpec`
- `internal/loomcli#driverSpec`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli#TestDriverSpec`
- `internal/loomcli/doc.go`
- `internal/orchcli#runsStrandAddSpec`
- `internal/orchcli#orchCLI.orchSpec`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`
- `internal/orchcli#TestStart_NoStrandLaunchesAndSpawnsWatcher`
- `internal/orchengine/doc.go`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#statusStrandShareNum`
- `internal/loomcli#statusStrandShareDen`
- `internal/loomcli#statusStrandMinRows`
- `internal/orchcli#runsStrandShareNum`
- `internal/orchcli#runsStrandShareDen`
- `internal/orchcli#runsStrandMinRows`

**Uses:**
- `plan:internal/reedengine/render#AnchorOverview`

**Intent:** Put the run's strands where D1 and D4 say: the two overview panes in the overview window, and the two coordinating agents in windows of their own.

`statusStrandAddSpec` (the loom-status strand running `lyx loom checklist`, used by both loom and darn runs through the shared bootstrap) and `runsStrandAddSpec` (the prime's `runs` strand running `lyx batten overview`) return a display of `render.AnchorOverview` alone, with no size and no segment, so the overview button takes the plain window-button style; their share and minimum-row constants go with the content size, and their doc comments, with the file comment at the head of `internal/orchcli/runs.go` ("the top content strand ... before the orchestrator strand"), say the strand is the session's overview window.
`driverSpec` changes the loom LLM driver strand from `below-parent` to `own-window`, keeping `Focus: false`, and its doc comment's anchor paragraph now says the driver is an agent window of its own, after the overview and before the shell, never hidden.
`orchSpec` changes the orchestrator strand from `below-parent` to `own-window` and keeps `Focus: true`, which card 7 makes select its window once opened.
A live session from an older `lyx` comes into the order through the fresh specs alone: `lyx loom start` and `lyx orch start` replace a status or runs strand launched by an older build (`statuspane.StrandCurrent`), which moves it to the overview window by card 6, and a new driver spawn opens its own window, while a resumed in-flight driver keeps its persisted `below-parent` place in the shell window until its run ends (D10).
The prime's operator now sees the batten overview or the orchestrator, one window each, not both at once, as the decision record's open risks accept.

Docs land here: `internal/loomcli/doc.go`'s status-strand paragraph says the strand is the overview window drawing card 2's frame with no content-size report; `internal/orchengine/doc.go`'s start and stop paragraphs say the `runs` strand is the overview window and the orchestrator has a window of its own, dropping "directly above the orchestrator" and "takes every row of the stack above Selvage"; the `orchCLI` start command's `Long` text in `start.go`, which `orchSpec`'s file holds, drops "above the orchestrator"; `shuttleengine.Spec`'s `Display` field comment and `internal/shuttleengine/doc.go` stop saying the orch and the loom driver stay in window 0, saying instead that a strand lands in a window of its own unless its spec names an anchor; `docs/overview.md`'s orch entry and its loom bootstrap passages say the runs and status strands are the overview window, the orchestrator and an LLM driver have windows of their own, and the session's windows run overview, agents, shell.

`TestStatusStrandAddSpec`'s "fields" subtest wants the overview display, and its "pinned by render" subtest is deleted, since render places neither an overview nor an own-window strand.
`TestDriverSpec`'s anchor row wants `render.AnchorOwnWindow`, with its message saying the driver's pane gets a window of its own and is never hidden; its focus row is unchanged.
`TestStart_EnsuresTheRunsStrandBeforeTheOrch`'s `wantDisplay` is the overview display.
`TestStart_NoStrandLaunchesAndSpawnsWatcher` wants `render.AnchorOwnWindow` with `Focus` still set.

**ImpactSummary:** Four strand specs in two CLIs change anchor and six content-size constants go; the specs' only callers are the loom shared bootstrap, the orch start verb and the two tests per CLI this card lists, and the remaining edits are docs.
