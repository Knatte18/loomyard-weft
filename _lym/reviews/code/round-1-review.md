---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/orchcli/status.go:172
    summary: The strand_down help text repeats every down-cause value as a string literal, which PATTERN-code-conventions' named-constants rule forbids; a renamed cause leaves the help silently wrong.
    fix: "In the Long string of statusCmd in internal/orchcli/status.go, end the raw literal after `strand_down names why the strand is not live, empty while it is:` and build the value list by concatenation from the constants, as the same string already does with orchKeyName: orchroute.DownNoStrandRecorded, orchroute.DownStrandNotInReedState, string(reedengine.DownPaneDead), string(reedengine.DownPaneUnlisted), string(reedengine.DownPaneElsewhere) and string(reedengine.DownStartShell), keeping the trailing `for the pane's start shell in the foreground.` wording. status.go already imports both orchroute and reedengine, and all six constants are exported."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/reedengine/apply.go#downCause
    summary: When the server-wide pane lookup fails, an own-window strand whose pane sits in another session reads down as pane-unlisted, so strand_down and the routing's log line misstate the one case the record wants made decisive; no assertion pins Down after a failed lookup.
    fix: "Have strandsElsewhereLocked in internal/reedengine/ownwindow.go also report whether its server-wide lookup failed, and pass that to downCause from Status in internal/reedengine/lifecycle.go. Add a DownCause constant beside the others in apply.go, such as DownPanePlacementUnread = \"pane-placement-unread\", returned for an own-window pane the session does not list when the lookup failed, before the pane-unlisted case. Name it in the strand_down help (through the constant, per F1). In TestStatus_ReportsOwnWindowStrandsFromTheSessionListing's failed-lookup loop, assert that strand gone reads the new cause and strand nowhere reads it too."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/reedengine/lifecycle_test.go#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing
    summary: The DownPaneUnlisted doc promises that cause for a strand bound to no pane, and no test row pins it.
    fix: "Add a strand with GUID \"unbound\" and PaneID \"\" (own-window display) to the fixture's Strands, and add \"unbound\" to wantDown as DownPaneUnlisted, to want as {own: true}, and to wantPanePID as 0."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/orchroute/live.go:66
    summary: "The log message \"orch: the orch reads as down\", which the change description names as the line to search for, is written as two separate literals in the Debug and Info branches of Live."
    fix: "Declare one unexported constant in internal/orchroute/live.go, such as const downReadingLogMessage = \"orch: the orch reads as down\", and use it in both the logger.Debug and the logger.Info call of Live."
---

### [BLOCKING] The strand_down help spells each cause as a literal

**Location:** `internal/orchcli/status.go:172`

**Issue:** The help lists `no-strand-recorded`, `strand-not-in-reed-state`, `pane-dead`, `pane-unlisted`, `pane-elsewhere` and `start-shell` as text inside the raw string.
Each value is already a named constant, in `internal/orchroute/live.go` and `internal/reedengine/apply.go`.
PATTERN-code-conventions says a name that can go stale is one named constant, referenced everywhere, never a repeated literal.
That makes this a constraint violation, which this review must rate BLOCKING.
The same `Long` string already renders config key names from code through `orchKeyName`, so the fix fits the verb's existing style.

### [LOW] A failed server-wide lookup turns pane-elsewhere into pane-unlisted

**Location:** `internal/reedengine/apply.go#downCause`

**Issue:** `strandsElsewhereLocked` answers nil when `list-panes -a` fails, so `downCause` sees an empty placement and returns `pane-unlisted` for a pane that may sit in another session.
The record asks for each down reading to name its cause so that the next occurrence is decisive, and pane-elsewhere against a vanished pane is the distinction it most needs.
The lookup failure logs one Warn per episode, but the Warn neither repeats nor appears in `strand_down` or the routing's line.
The failed-lookup loop in the test asserts only `Elsewhere`, so it does not pin `Down`; the tests lens raised that same gap, merged here.

### [LOW] No row pins the cause for a strand bound to no pane

**Location:** `internal/reedengine/lifecycle_test.go#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`

**Issue:** The doc of `DownPaneUnlisted` covers a strand "bound to no pane", and `downCause` yields it for an empty pane id, because `presentIDs[""]` is false and `strandsElsewhereLocked` skips an empty `PaneID`.
The fixture has no strand with an empty `PaneID`, so a reordering of the switch that broke this case would pass.

### [LOW] The down-reading log message is two literals

**Location:** `internal/orchroute/live.go:66`

**Issue:** The Debug and Info branches each spell `"orch: the orch reads as down"`.
The change description tells the operator to look for this line, so the two branches must not drift apart.

## Rejected

- No test of `Live`'s log levels (code-tests lens): the log line is output a human reads, and the testing rules give such output no test.
- The help omitting `no-reed-session`: correct, since `lym orch status` returns its error envelope from `Strands()` before it reaches `ReadingOf`, and that error names the missing session.
- `strand_live` and `strand_down` disagreeing: both come from the same first-read state and strand list, and `restartWatcher`'s re-read never feeds the envelope.
- `no-strand-recorded` logged at Debug: the record's list of causes the log must name does not include a never-started orch, so Debug does not breach it.
- Info-line cost on the routing paths: `Live` is called once per chain step of a notice or decision check, not from a polling loop, and `Destination` skips it for a start worktree whose strand reed holds.
- Merge danger `two-way`, blast radius `orch`: the diff adds a JSON key and an in-memory struct field, with no record, config meaning or outward push changed; `Live`'s return value is unchanged for every caller.
- Root-Cause trailer: under the amended record, which settles on naming the cause instead of fixing #232, the trailer names the defect the diff removes, a down reading with no cause.
