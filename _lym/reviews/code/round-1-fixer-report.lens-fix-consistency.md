```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F1–F3 change set (commit `a1e591ec3`) agrees with its surroundings.

no findings

- The one production caller, `internal/orchcli/remotecontrol.go`, only prints the error text after `lym: ` and matches no string, so the new unreadable-row error needs no change there.
- The new message follows the form of its sibling errors (`reed: client %q …`).
- The doc comments on `ClientAgentPane`, `clientAgentPaneVia` and `TestClientAgentPaneVia` still hold.
- The grep showed no other copy of the changed message or row format.
- The integration test reads only success cases, so the change does not affect its expectations.

Tool calls:
- `git show --stat` plus a grep for callers and message strings: my context lacked the commit's file list and the `ClientAgentPane` caller.
- `sed` read of `internal/orchcli/remotecontrol.go`, lines 185–215: my context lacked how the caller handles the error.
