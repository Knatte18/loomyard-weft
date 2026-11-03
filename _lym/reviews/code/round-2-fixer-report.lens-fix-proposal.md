```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F2, F3 and F4 proposals, which the fixer carried out as proposed. F1 was fixed with a different change (no Topology field), so I left it out.

**Findings**

1. **MEDIUM** — F4's proposal drops the weft-root failure from the in-process verb result.
   - Changed text: `runRemoteTeardown` now sets `out.weftRootErr` where it used to set `out.err` (`internal/fabricengine/remoteteardown.go`).
   - Summary: the in-process callers still return only `out.err`, so a weft-root failure now reads as success.
   - Evidence:
     - `remove.go` ends with `return res, out.err`.
     - `taskbranchdiscard.go` has `return res, out.err`.
     - `cleanup.go:321` tests `out.err != nil` to set `entry.Error`.
     - None of these read `weftRootErr`.
   - Case: a `Remove` that runs the job in process (`inProcessHold`, which every test binary uses) and cannot resolve the records repo after a successful archive. The verb used to return `resolve weft repo root: …`. It now returns a nil error with no `RemoteBranchError`. The weft delete and task-branch delete never ran, and nothing reports it.
   - Scope: production hands the job to the detached child, and the child's notice does report `weftRootErr`. Only the in-process path is affected. A test that pinned the old error would fail, and none was added for the new path.

I found nothing against F2 or F3.
- F2's notices do reach the creel in the child: `cmd/lym/main.go` installs `orchcli.InstallPushNotifier` for every non-test process.
- F3's wording follows `unknownKindWayForward`, and the test asserts all three alternatives.

**Tool calls**
- grep of `outcome.err`, `remoteOutcome`, `handedOff` and `.ran` across `internal/fabricengine`, plus `cmd/lym/main.go` lines 145–165: my context had no callers of `remoteOutcome` and no notifier-install site.
- `cleanup.go` lines 312–345, `taskbranchdiscard.go` lines 60–85 and `remove.go` lines 376–410: my context lacked how the in-process callers consume `out.err`.
