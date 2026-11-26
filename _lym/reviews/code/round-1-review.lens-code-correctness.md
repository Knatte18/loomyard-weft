```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I checked each branch the diff changes in `clientAgentPaneVia` against a concrete input, plus the fake-tmux path of the rc-on verb test.

**Function or branch, case run, result**

- `clientAgentPaneVia`, `list-clients` fails (no server): returns `reed: read the active pane of client "/dev/pts/3": <err>`. Correct: the error is wrapped and names the client.
- `clientAgentPaneVia`, output empty (tmux exits 0 with no attached clients): `Split("", "\n")` gives `[""]`, a single field, so the row is skipped and the function returns `no client "/dev/pts/3" is attached`. Correct.
- `clientAgentPaneVia`, several clients listed and the clicker's row in the middle (`/dev/pts/1`, `/dev/pts/3`, `/dev/pts/30`): the match is exact on field 0, so `/dev/pts/30` is not taken for `/dev/pts/3`. Returns `{%7, scout, false}`. Correct.
- `clientAgentPaneVia`, matching row with `@lym_rc` = `1`: `RemoteControlOn` is true. Correct.
- `clientAgentPaneVia`, matching row with an empty fourth field: `RemoteControlOn` is false. Correct.
- `clientAgentPaneVia`, CRLF line endings: `TrimRight(line, "\r")` runs per line, so the last field stays clean. Correct.
- `clientAgentPaneVia`, matching row with an empty pane id: returns `shows no active pane`. Correct, though real tmux cannot produce it: `list-clients` only lists clients that have a session.
- `clientAgentPaneVia`, the clicker's row has 3 fields or 5 (an option value with an embedded tab): the row is skipped and the error says `no client "<name>" is attached`. That message is false, because the client is attached and listed; its row just didn't parse. Real output can't produce this as long as strand names and `@lym_rc` hold no tab. See F1.
- `clientAgentPaneVia`, an option value with an embedded newline: the row splits into two fragments, neither has 4 fields with field 0 equal to the client, and the result is `no client attached`. Same mislabel as F1, and also unreachable for a valid strand name.
- `clientAgentPaneVia`, `client` is the empty string (required flag given as `--client ""`): no real row has an empty `client_name`, so it returns `no client "" is attached`. Correct.
- Format expansion: tmux's `list-clients` sets up each row's format from that client alone, so `pane_id` and the pane user options resolve to `c->session->curw->window->active`. That is the clicking client's own active pane. Correct, and `io.go:197` already relies on `list-clients -F #{pane_id}` being per client.
- Client-name round-trip: the click line passes `--client #{q:client_name}` (`panebin.go:288`), and the shell unquotes the escaping `q:` adds. Field 0 is the raw `#{client_name}`, so the two compare equal. Correct.
- orchcli `TestRemoteControlOnVerb_QueuesForTheClientsPaneWithoutMarkingIt` fake tmux: the pattern `*'list-clients -F '*` matches the argv `... list-clients -F #{client_name}...`. `printf` emits `/dev/pts/3\t%7\ttst:orch\t\n`, which is 4 fields matching `--client /dev/pts/3`. The failure branch exits 1 and the test sees the wrapped error. Correct.

**Findings**

- **F1.** LOW, class `design`, at `internal/reedengine/remotecontrol.go:49` and `:57`. A row for the clicking client that has the wrong field count is skipped like another client's row. It then gets the error `no client "<name>" is attached`, which is false for a client tmux did list. The `TooFewFields` row in `internal/reedengine/remotecontrol_test.go` (diff `002`, line 150) asserts that false message for a row naming `/dev/pts/3`. Evidence: `remotecontrol.go:49` (`len(fields) != 4 || fields[0] != client` → `continue`) and `:57`. Fix: match on field 0 first. If that row then has the wrong field count, report it as unreadable, for example `shows no readable pane row`, and correct the `TooFewFields` row's expected error.

**Tool calls**

- `sed -n 140,215p internal/orchcli/remotecontrol_test.go`: the rc-on verb test body. My context held only its diff hunk, so I read it to confirm the `--client` value matches the fake row and to check the case pattern.
