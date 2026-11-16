```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for consistency with its surroundings (docs, help text, twin call sites, tests).

No findings.

I checked these places and found no disagreement:
- **`ReadRemoteControl` and `RemoteControlOn`:** the only non-test callers are `remotecontrol.go` and the adopt carry-over in `start.go`. Their doc comments match the new behavior.
- **`rc-bar` text:** the `rc-bar` help, `orchcli/doc.go`, `remoteControlBarWordAt`'s comment and the `NoRecord` and `RecordedOnForAnOldStrand` rows all say a strand with no record prints nothing. `reedengine/doc.go` only says the button is drawn while the state line holds lit or unlit, which still holds.
- **`rc-on` text:** its help, `orchcli/doc.go` and `turnOnRemoteControl`'s comment agree on the refusals (no live strand, several strands, no engine support, already on). The way-forward wording matches the F2 ruling.
- **`recordRemoteControl`:** it now skips an engine with no remote control. The launch and adopt call sites, the click, `TestStart`'s `EngineWithoutRemoteControl` row and the docs agree on that.
- **`cli.go`:** the group help and the nearby comment agree on `rc-bar` being exempt.
- **Doc cross-references:** `orchengine/doc.go` and `bindings.go` don't touch the changed behavior, so none is stale.

One unremarkable side effect of the F8 fix: an orch launched before this change has no record, so its RC button is no longer drawn until it is relaunched. The review's own fix asked for this and the docs state it, so I did not count it as a finding.

Tool calls:
1. `grep` across non-test `.go` and `.md` files for `RemoteControlOn`, `ReadRemoteControl`, `rc-bar`, `rc-on` and "RC button". I used it to find every doc or caller outside the files I already held.
