```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. Sets rerun: F6 (presence-watch mentions), F3 (`RootProjectHandle` references), F1 (relay lifetime docs and no-button relay tests), F2/F5 (`retire` and `dropForwarding` callers).

**LOW — stale `spareOrch` comment** (`internal/slackengine/relay_test.go:33-35`)
- The comment says "Pressing its button wakes the relay and makes no channel."
- After the F2/F5 change, the first `retire` pass drops that button, since the beam is unlisted and has no channel entry.
- The only remaining use is `relay_test.go:913`, where the button is on at the start and the test asserts only the created channels.
- The comment is now untrue. It should say the button is dropped by the first retire and makes no channel.

Set checks that held:
- **F6:** no other `.md` or `.go` file names the presence watch for the relay. The fix covers the whole set.
- **F3:** `RootProjectHandle` has no references left in `.go` or `.md` files.
- **F1:** no relay test starts with every button off before the new row (`relay_test.go:911` keeps the prime on). `docs/overview.md:898`, `slackcli/cli.go:136` and `start.go:113` still hold, since the relay ending when no button is left on agrees with them.
- **F2/F5:** the only production caller of `keeper.retire` is `relay.go:658`. The only production caller of `dropForwarding` is the retire loop at `keeper.go:442`, so nothing else needed the change.

Unchecked: F4, F7, F8 and F9 name no set to rerun.

Tool calls:
1. Grep for `presence watch`, `RootProjectHandle`, relay-lifetime wording and `spareOrch`. My context lacked a repo-wide search of docs and Go files.
2. Grep for the `retire` and `dropForwarding` callers, and `setButton(...false)` in the relay tests. My context lacked a repo-wide caller search.
