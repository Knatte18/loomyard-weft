# Card 4 — a paused verify marker reads as no activity and shows as paused

**Edit:**
- `internal/loomcli#agentActivityFor`
- `internal/loomcli#verifyWaiting`
- `internal/loomcli#TestVerifyWaiting`
- `internal/loomcli#TestAgentActivity_LiveLockWaitReadsAsActivity`
- `internal/battencli#readAgentActivity`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`

**Uses:**
- `plan:internal/verifytree#MarkerStatePaused`
- `internal/verifytree#ReadMarker`

**Intent:**
D4's activity readers.
A Verify held by a pause does no work, so `agentActivityFor` and `readAgentActivity` read a live marker whose state is `verifytree.MarkerStatePaused` as no activity and fall through to their parent lock, wait marker and run readings, exactly as with no marker; a running or waiting marker still reads as activity now.
The loop watchdog already skips its idle timeout while the step is covered, and batten's quiet notice treats a paused Verify as it treats a held run.
`verifyWaiting`, loom status's note, renders a paused marker as `<producer>: verify held by a pause <elapsed> (<site>)`, elapsed from the marker's `WaitStarted`; the running and waiting notes are unchanged.
Their doc comments name the paused state.

`TestVerifyWaiting` gains a row with a live paused marker and its exact note.
`TestAgentActivity_LiveLockWaitReadsAsActivity` is already the table over `agentActivityFor`; it gains a setup column, its rows becoming the lock-wait record or a verify marker written under the location's verify directory, and three marker rows: a live running marker reads as activity now, a live paused marker reads as none, a dead marker reads as none.
Its doc comment says it covers the verify marker as well as the lock record.
`TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` gains the same two marker checks, running then paused, before its lock-wait checks.

**ImpactSummary:** three readers of the verify marker in loomcli and battencli change one branch each; their callers (loop watchdog, batten quiet notice, loom status) get no activity from a paused Verify.
