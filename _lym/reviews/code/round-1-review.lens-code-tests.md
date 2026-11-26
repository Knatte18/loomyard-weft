```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviour → pinning test**

1. **list-clients argv and four-field format.** `TestClientAgentPaneVia` checks `wantArgv` on every row (`internal/reedengine/remotecontrol_test.go:55`). Pinned.
2. **Mapping a row to `AgentPane`** (pane id, strand name, mark set or unset). Rows `AgentPaneWithTheMark`, `AgentPaneWithoutTheMark` and `PaneWithNoStrandName`. Pinned.
3. **Other clients' rows are skipped.** Row `OtherClientsRowsAreSkipped`. Pinned only in part: the match on `client_name` must be exact, and no row would fail if it were a prefix match. See finding 1.
4. **Listed client with an empty pane id gives "shows no active pane".** Row `EmptyPaneID`. Pinned.
5. **Unlisted client, or empty output, gives `no client "<name>" is attached`.** Rows `ClientNotListed` and `EmptyOutput`. Pinned.
6. **A row with the wrong field count is skipped and the read falls through to the not-attached error.** Row `TooFewFields`. Pinned.
7. **A tmux failure is wrapped and names the client.** Row `TmuxError`, plus orchcli `UnreadablePaneNamesTheClient` at the verb level. Pinned.
8. **A trailing `\r` is trimmed from each row.** No row has `\r` in its output. See finding 2.
9. **Each client reads its own session's pane, live.** `TestClientAgentPane_ReadsEachClientsOwnSession` (tier 2) covers it.
   - On the old `display-message -c` read, both clients resolve to one session, so one of the two reads fails whichever client attached last.
   - Each client's name comes from a session-scoped `list-clients` inside `waitUntil`, and every read happens after both attaches. So the attach and the read do not race.
   - It is pinned.
10. **The rc-on verb queues into the clicking client's pane, which it reads through list-clients.** orchcli `AgentPaneIsQueuedAndLeftUnmarked` (`internal/orchcli/remotecontrol_test.go:158`): the fake answers only `list-clients -F` with the client's row. Pinned.

**Findings**

1. **LOW, design.** Location: `internal/reedengine/remotecontrol_test.go`, row `OtherClientsRowsAreSkipped`.
   - The row lists `/dev/pts/3` before `/dev/pts/30`.
   - A mutant that matches `client_name` by prefix (`strings.HasPrefix(fields[0], client)`) still returns the `/dev/pts/3` row and passes.
   - The exact match at `internal/reedengine/remotecontrol.go:49` (`fields[0] != client`) is therefore not pinned against a prefix match.
   - Fix: put the `/dev/pts/30` row before the clicking client's row.
2. **NIT, design.** Location: `internal/reedengine/remotecontrol_test.go`, `TestClientAgentPaneVia` rows.
   - No row ends a line with `\r\n`.
   - Removing `strings.TrimRight(line, "\r")` (`internal/reedengine/remotecontrol.go:48`) would leave `"\r"` as the mark field and read `RemoteControlOn` true, and no test would fail.
   - The old code's `\r\n` trim was untested too, so this is an inherited gap carried into the new per-row loop.

**Tool calls**

- Read `internal/orchcli/remotecontrol_test.go:130-230`, the verb test's assertions; the diff showed only its fake-tmux hunk.
- `sed -n 150,200p internal/reedengine/remotecontrol_test.go`, which returned nothing: the range was past the end of the file.
- Read `internal/reedengine/remotecontrol_test.go:50-80`, the `TestClientAgentPaneVia` assertion block; the diff showed only its table.
