```yaml
lens: fix-consistency
outcome: reported
```

Findings for the fix-consistency lens:

**LOW — the `bridge-unsettled` paragraph in the creel package doc is stale.**
- Changed text: the `bridge-unsettled` case added to `machineHoldWayForward` (`internal/creelengine/machinemail.go`).
- `internal/creelengine/doc.go:824-825` says the return carries `bridge-unsettled` when the relay reported no hold, and that its notice "names what clears the reason".
- That sentence was not true for `bridge-unsettled` before the fix, and it is true now. The paragraph does not say what clears it (a slack button of the hub turned on so the relay runs).
- This is not an inconsistency, only a missed chance to say it. I would leave it.

**LOW — relay start-up still settles accepted posts separately.**
- Changed text: `bridgeSender.send` in `internal/slackengine/bridgeout.go`, which now settles accepted posts first.
- `internal/slackengine/relay.go:249` still calls `sender.settleAccepted()` once at relay start, before the first `send`.
- The call is now redundant: the first send does the same. It also does one thing the send does not, which is to report the failure under the separate `bridge-settle` health key.
- This is not wrong, so it is not a defect, only a duplicate the fixer did not mention.
- The test `RelaySettlesAcceptedMessagesBeforeAnyTake` still passes.

No finding on the other twins I checked:
- No stale uses of `maxMachineNameLength`, `machineEnvelopeLead` or `maxNameLength` remain.
- `noticeReturnedLocked` has two callers and both pass the new argument.
- No `strings.Cut` or `strings.Split` on the machine address separator remains outside `internal/agentname`, which covers the finding's second site (`machineHoldWayForward`).

Calls made:
1. grep for the renamed or removed symbols and the `settleAccepted` callers. My context lacked a repo-wide view of stale references.
2. Read `internal/creelengine/doc.go:815-835` and `internal/slackengine/relay.go:240-256`, to judge the two hits above.
3. grep of `docs/slack-app.md` and `contracts/orchrole/role-slack.md` for wording on the name-taken retry, for stale sibling docs (no match).
