# Card 5 — The network gate leaves a waiting marker while it waits, and one process may pass every entry

**Create:**
- `internal/pauserecord#WaitingMarker`
- `internal/pauserecord#WaitingDir`
- `internal/pauserecord#ListWaiting`
- `internal/pauserecord#SetGateExemptAll`

**Edit:**
- `internal/pauserecord#Gate`
- `internal/pauserecord#Network`
- `internal/pauserecord#TestNetwork`
- `internal/pauserecord/doc.go`

**Uses:**
- `internal/pauserecord#ListNetOps`
- `internal/pauserecord#NetDir`
- `internal/pauserecord#InstallGate`

**Intent:**
Two gate additions D5 reads.

Waiting marker.
`Network` writes a `WaitingMarker` (`PID`, `StartTime`, `Slug`, from the gate) to `<WaitingDir>/<pid>-<n>.json` before it calls `WaitUncovered`, and removes it as soon as the wait returns, before it writes its in-flight marker.
`WaitingDir` is the `waiting` directory beside the record, a sibling of `NetDir` and never inside it, so `ListNetOps`, and so hibernate's in-flight reading, never lists a waiting marker.
`ListWaiting` lists the markers whose process `alive` reports live, removing and skipping the rest, as `ListNetOps` does, so a marker of a dead process or of a reused pid reads absent.
A marker that cannot be written is skipped; the gate's contract that it only waits and fails no call holds, and no gate decision reads the marker (`PATTERN-pause-holds-prompts`).

Exempt all.
`Gate` gains `ExemptAll bool`; with it set, `Network`'s waiter lets every live entry pass.
`SetGateExemptAll` sets it on the installed gate and returns the function that restores the previous value, doing nothing when no gate is installed, as `SetGateExempt` does.
It serves `lyx fabric remove` (card 11): an explicit removal is the operator acting through the pause, and its bound is the one process: a detached child installs its own gate in its own process, so the exemption never reaches a child.

`TestNetwork` gains rows: while a covering entry holds the call, `ListWaiting` lists one marker with the gate's pid, start time and slug and `ListNetOps` lists none; once the entry is lifted and the call is past, `ListWaiting` lists none and `ListNetOps` lists the in-flight marker; a waiting marker whose `alive` reports dead is listed by neither and its file is removed; with `SetGateExemptAll` a call covered by a hub entry and by a slug entry passes at once without sleeping, and its restore puts back the holding behaviour; `SetGateExemptAll` with no gate installed installs none and its restore is a no-op; a gate whose waiting directory path is taken by a plain file still waits and then passes the call once the entry lifts, with no marker listed.
The package doc gains both, and that the marker records only what the gate waits on.

**ImpactSummary:** additive fields and functions in pauserecord; Network's signature and every existing caller are unchanged.

**Commit:** 5: the network gate leaves a waiting marker, and SetGateExemptAll
