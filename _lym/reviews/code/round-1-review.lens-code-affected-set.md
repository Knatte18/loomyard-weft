```yaml
lens: code-affected-set
outcome: reported
```

Changes and the affected set I found for each:

1. **`clientAgentPaneVia` now runs `list-clients -F` instead of `display-message -p -c`** (`internal/reedengine/remotecontrol.go:41-58`).
   - The only production caller, `remoteControlOnCmd`, is unchanged in signature and covered (`internal/orchcli/remotecontrol.go:195`).
   - The fake tmux in `TestRemoteControlOnVerb_QueuesForTheClientsPaneWithoutMarkingIt` was updated to answer `list-clients -F` (`internal/orchcli/remotecontrol_test.go:165-169`). Its doc comment ("a client whose active pane cannot be read fails the verb naming the client") still holds. Covered.
   - The tier-0 table in `TestClientAgentPaneVia` was updated. Covered.
   - `panebin.go`'s `composeRemoteControlClickCommand` and its test `panebin_test.go:521,529` still pass `--client #{q:client_name}`. The shell unquotes that back to the raw `#{client_name}`, which is the value the `fields[0] != client` match compares against. Covered, no change needed.
   - In the multiplexer contract doc, `internal/reedengine/doc.go:222-224` says `list-clients` is absent from `requiredSubcommands` because "Geometry option pins are non-fatal". The RC click now depends on `list-clients` too, so that one reason no longer covers every use. On a multiplexer without `list-clients`, the click fails at click time and the capability probe never catches it, whereas `display-message` is required. Finding 1.
   - `probe.go` `requiredSubcommands` (`internal/reedengine/probe.go:32-47`) is unchanged. It is the same set as the bullet above and is judged in finding 1; I raise no separate finding.
   - `internal/toolfloor/doc.go` feature list: `list-clients -F` and `#{client_name}` predate tmux 3.0, so the list needs no entry. Covered.
   - `docs/`, `tools/sandbox/SANDBOX-REED-SUITE.md` and `docs/reference/tmux_scripting.md` hold no description of the RC read. Nothing stale.
2. **New error text `reed: no client %q is attached`, and the too-few-fields case moves to it from `shows no active pane`.**
   - Outside the diff's own tests, no test, doc or stencil asserts `shows no active pane` or the new text. The grep hit `attach_test.go:439-472` asserts a different message ("another client is attached"). Covered.
3. **Commit messages and departures.**
   - `d14e9c692` carries `Root-Cause:` naming the defect the diff removes, a format pane-resolved from the most recently active client, not the `-c` client. The trailer is valid.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer appears. The loosened-check bullet does not apply: the `TestClientAgentPaneVia` rows change because the argv changed, not because a tolerance was widened.
4. **Declared PATTERN changes:** the directive declares none, so nothing is checked.
5. **Rule documents:**
   - `PATTERN-test-speed`: the new tier-2 test sits in its own `_integration_test.go` file under `//go:build linux` (a platform term, allowed). It opens with `tiergate.Require(t, 2)` and `t.Parallel()`. Covered.
   - `PATTERN-test-isolation`: the test uses `newIntegrationEngine` and the package `TestMain` (`testmain_test.go:13`). Covered.
   - `PATTERN-reed-panes`: no pane typing is touched. Not affected.

Findings:

1. Severity LOW, class consistency, location `internal/reedengine/doc.go:222-224`. The contract doc gives "geometry option pins are non-fatal" as the reason `list-clients` is absent from `requiredSubcommands`, but `list-clients` is now also the RC click's pane read. A multiplexer lacking it fails every RC click at click time and passes the capability probe, a case the doc's stated reason no longer covers. Fix: amend the reason to name the RC click's read and why it stays optional, or add `list-clients` to `requiredSubcommands`. Evidence: `internal/reedengine/doc.go:222-224`, `internal/reedengine/probe.go:32-47`, `internal/reedengine/remotecontrol.go:43`.

Outside my lens, for the handler: the new doc comment on `clientAgentPaneVia` (`remotecontrol.go:39-40`) explains the change by the rejected `display-message` alternative. That may conflict with code-quality's ban on rejected-alternative trails in doc comments.

Tool calls:
- A grep across the repo for readers of `shows no active pane`, the new message, `rc-on`, `RemoteControlOnVerb`, `ClientAgentPane` and `clientAgentPaneVia`: my context lacked a repo-wide reader search.
- A read of `internal/orchcli/remotecontrol_test.go:140-210` plus a grep of the RC and `display-message` docs: my context held only the diff hunk of that test, not its rows and doc comment, and had no grep of the RC docs.
