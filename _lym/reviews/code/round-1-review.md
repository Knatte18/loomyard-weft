---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/reedengine/remotecontrol.go:49
    summary: a listed row for the clicking client with the wrong field count is skipped and reported as the false error that no such client is attached
    fix: "In clientAgentPaneVia, split each line, then skip only rows whose first field is not client. For the client's own row, return an error naming the row as unreadable (for example `reed: client %q has an unreadable list-clients row`) when len(fields) != 4, before the empty-pane-id check. In TestClientAgentPaneVia (internal/reedengine/remotecontrol_test.go) change the TooFewFields row's wantErr to that new text."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/reedengine/remotecontrol_test.go#TestClientAgentPaneVia
    summary: the OtherClientsRowsAreSkipped row lists /dev/pts/3 before /dev/pts/30, so a prefix match on client_name would still pass and the exact match is unpinned
    fix: "In the OtherClientsRowsAreSkipped row's out, move the `/dev/pts/30\\t%9\\tother\\t1\\n` line before the `/dev/pts/3\\t%7\\tscout\\t\\n` line, keeping the want unchanged."
    origin: lens:code-tests
  - id: F3
    severity: NIT
    class: design
    location: internal/reedengine/remotecontrol_test.go#TestClientAgentPaneVia
    summary: no row feeds a CRLF line ending, so removing the per-row carriage-return trim breaks no test and would read the mark as on
    fix: "Change the AgentPaneWithoutTheMark row's out to `/dev/pts/3\\t%7\\tscout\\t\\r\\n`, keeping want as AgentPane{PaneID: \"%7\", StrandName: \"scout\"}."
    origin: lens:code-tests
---

The diff makes the change the board entry asks for.
`clientAgentPaneVia` now reads the clicking client's row of `list-clients -F`, whose format tmux expands against that client's own session.
That is the entry's "resolve the client's session first" route in one call, and `summary.md` records why it was chosen over `-t`.
The `Root-Cause:` trailer on `d14e9c692` names the defect the diff removes.
The tier-2 test attaches one client per session before either read, so the reads do not race the attaches.
It sits in a linux-only file at one tier and registers its server through `newIntegrationEngine`'s `KillOnCleanup`.
The sibling `display-message -c` reads in `switch.go` and `buttonclick.go` are out of the entry's scope, and board note `reed-switch-wrong-client` exists as `summary.md` says.
The `two-way` door holds: the diff changes no record, board format or config meaning and pushes nothing outward.

### [LOW] Wrong-field-count row is reported as an unattached client

**Location:** internal/reedengine/remotecontrol.go:49
**Issue:** `len(fields) != 4 || fields[0] != client` skips the clicking client's own malformed row exactly like another client's row, so the read ends at `no client %q is attached`, a false statement for a client tmux listed. The `TooFewFields` row pins that false text. Real tmux output reaches this only if a pane option holds a tab or newline, hence LOW.

### [LOW] Exact client-name match is not pinned against a prefix match

**Location:** internal/reedengine/remotecontrol_test.go#TestClientAgentPaneVia
**Issue:** In `OtherClientsRowsAreSkipped` the clicking client's row `/dev/pts/3` comes before `/dev/pts/30`, so a `strings.HasPrefix` mutant of the match at `remotecontrol.go:49` returns the right row and passes. The row's purpose, other clients' rows are skipped, is only half checked.

### [NIT] Per-row carriage-return trim is uncovered

**Location:** internal/reedengine/remotecontrol_test.go#TestClientAgentPaneVia
**Issue:** No row ends a line with `\r\n`. Dropping `strings.TrimRight(line, "\r")` at `remotecontrol.go:48` would leave `"\r"` as the fourth field and read `RemoteControlOn` as true, with no failing test.

## Rejected

- code-affected-set's `doc.go:222-224` finding: the bullet explains why `list-clients` is optional for geometry pins and already did not cover `io.go:197`'s per-client read. A multiplexer without `list-clients` makes the RC click fail with a wrapped error on the clicking client, so no silent wrong behaviour follows from the doc. The finding is a pre-existing doc gap, not one the diff forces.
- The suggestion that `clientAgentPaneVia`'s doc comment carries a rejected-alternative trail: its lines name a tmux trap, that `display-message` from outside tmux expands against the most recently active client. That is the non-obvious WHY of the list-clients read, which a maintainer needs so they don't swap it back, and not a history of rejected designs.
