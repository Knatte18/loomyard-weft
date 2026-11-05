# Card 11 — orch-relay-presence

**Create:**
- `plan:internal/orchengine#WithPresence` -> `func WithPresence(p Paths, pointer string) string`
- `plan:internal/orchcli#orchCLI.startRelayStep` -> `func (c *orchCLI) startRelayStep(report *BeamUpReport)`
- `plan:internal/orchcli#relayStarter` -> `func relayStarter(location *lymcwd.Location, beam bool) func() (slackengine.StartOutcome, error)`

**Edit:**
- `internal/orchengine#Paths`
- `internal/orchengine#ResumeContext`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.startAutoReload`
- `internal/orchengine#Watcher.reloadAfterCompaction`
- `internal/orchengine#TestResumeContext_RendersPointerPerPhaseAndWritesMark`
- `internal/orchengine#TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped`
- `internal/orchengine#TestWatcher_HandoffCompleteClearsThenResumes`
- `internal/orchengine#TestCompact_RequestedCycleClearsRequestAndCompactsAfterNote`
- `internal/orchcli#orchCLI`
- `internal/orchcli#wireCLI`
- `internal/orchcli#orchCLI.beamUp`
- `internal/orchcli#orchCLI.launchOrch`
- `internal/orchcli#orchPaths`
- `internal/orchcli#TestStart_StartsTheDobbiesFirst`
- `internal/orchcli#TestStart_ResumesTheRecordedSession`
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Uses:**
- `internal/orchcli#BeamUpReport`
- `internal/orchcli#newStartHarness`
- `internal/slackengine#ReadPresence`
- `plan:internal/slackengine#EnsureRelay`
- `internal/hubgeom#SlackGeometry`

**Intent:** Bring the relay up with the prime orch, and carry the presence in the orch's resume pointer (D3, D6).

Relay start at the prime's bring-up (D3):
- `orchCLI` gains `startRelay func() (slackengine.StartOutcome, error)`, which `wireCLI` fills with `relayStarter(location, beam)`: nil for a beam, and for the prime a closure that builds `hubgeom.SlackGeometry(location.HubPath)` and calls `slackengine.EnsureRelay` with `location.WorktreeName`, the prime's name.
- `beamUp` calls `startRelayStep` right after `startDobbyStep`; a nil seam starts nothing; `not-configured`, `live` and `started` add nothing to the report; an error only warns, "the Slack relay was not started (<cause>); way forward: run `lym slack start`", and the bring-up goes on, since the relay is no part of the orch.
  So `lym beam start` of the prime, a fresh bring-up and the anchor change's restart of the prime through `lym beam start` all start a down relay of a configured hub.
- `internal/orchcli/doc.go`'s list of bring-up steps gains the relay step after the dobby step, with its warning-only failure and way forward.
- `TestStart_StartsTheDobbiesFirst` gains rows: with a relay seam, the prime start calls it once, after the dobby and before the strand; a relay seam error warns naming `lym slack start` and still launches the strand; a nil relay seam calls nothing; `relayStarter` with `beam` true answers nil and with `beam` false answers a non-nil starter.

The presence line (D6):
- `Paths` gains the field `SlackPresencePath`, which `orchPaths`, the constructor every orch verb and the watcher take their paths from (`PrimePaths` delegates to it), sets from `hubgeom.SlackGeometry(location.HubPath).PresencePath`; a geometry error leaves it empty, and an empty path reads as no line. `orchroute.Paths`, which `orchPaths` wraps, stays without it: its other callers read orch state and render no pointer.
- `WithPresence` returns pointer unchanged unless `slackengine.ReadPresence(p.SlackPresencePath)` reads `slack`, and then appends, on the same line since a typed pointer is one creel text line, " Presence is slack: also post every message meant for the operator to the hub's SLACK program, as the orch role's Slack rules say." A read error logs at Debug and leaves the pointer unchanged.
- Every rendered pointer passes through `WithPresence` once, where it is rendered: `ResumeContext`'s rendered resume and reload pointers (the `lym orch resume-context` hook), `Watcher.startClearing`'s and `Watcher.reloadAfterCompaction`'s resume pointers, `Watcher.startAutoReload`'s reload pointer, and in `orchCLI.launchOrch` both the reload pointer of a resumed session and the prompt `orchengine.ChooseStartPrompt` picks for a fresh launch, before any note is appended.
  `internal/orchengine/doc.go`, which calls the reload pointer the one-line stencil, says the rendered pointer may carry the presence line appended to it.
  `ResumeContext`'s verbatim branch returns `st.PendingResume` as stored, since `startClearing` stored it with the line already, so the mark equals the pending pointer and the watcher's hook-delivered check still skips typing it again.
- `TestResumeContext_RendersPointerPerPhaseAndWritesMark` gains rows: with a presence file reading `slack`, the reload and resume pointers end with the presence line once and the mark carries it; with `terminal`, and with no file, they carry none; a resuming phase with a pending pointer that carries the line returns it unchanged, the line once.
- `TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped` gains a row: with presence `slack`, the typed reload pointer is the rendered pointer with the presence line.
- `TestWatcher_HandoffCompleteClearsThenResumes` gains a row: with presence `slack`, the stored and typed resume pointer carries the line once.
- `TestCompact_RequestedCycleClearsRequestAndCompactsAfterNote` gains a row: with presence `slack`, the resume pointer typed after the compact cycle's note carries the line.
- `TestStart_ResumesTheRecordedSession` gains rows: with presence `slack`, the resumed session's reload pointer and a fresh launch's prompt from the last note each carry the line.

**ImpactSummary:** One new field on Paths and one on orchCLI, both empty by default so every existing orch test keeps its expected pointer; the bring-up gains one warning-only step for the prime, which `orchcli/doc.go`'s bring-up steps name, and `orchengine/doc.go` says the pointer may carry the presence line.
