---
round: 2
focus:
  - "internal/fabricengine/remoteteardown.go `noticeRemoteTeardown` and `internal/fabricengine/remove.go` `deleteTaskBranchOnOrigin`: settle whether every failed remote step of the teardown child (weft push, archive tag push, task-branch ls-remote, fetch or delete, weft-branch delete) reaches both the fabric Warn log and one creel notice naming slug, operation and way forward, and whether more than the first failing step is reported."
  - "internal/fabriccli/teardownchild.go `teardownChildCmd`, the pause exemption set around `fabric.go` removal, and the paragraph in internal/fabriccli/doc.go on detached children: settle whether the teardown child, which runs the remote steps while holding the exclusive teardown lock, is consistent with the removal's pause exemption, and whether the docs and the lock refusal text state which holds."
  - "internal/fabricengine/remoteteardown_test.go and removefields_test.go: settle whether `RunTeardownChild` running a job, `spawnTeardownHold`, `childHold.finish`, `RemoteHandedOff` and `noticeRemoteTeardown` each have a row that would fail if the step order (weft tip, archive tag, leased delete), the lock held to the last step, or the notice text broke."
  - "internal/fabricengine/remoteteardown.go `spawnTeardownHold` and the `runproc.EndSparing` call in internal/pairteardown/teardown.go: settle whether a later teardown or tender end of the same slug can end a running teardown child mid-job without a trace."
  - "internal/fabricengine/remoteteardown.go default way forward and `RunTeardownChild` job read: settle whether the way forward fits the job kind (remove or cleanup) and whether an unreadable job leaves a notice."
  - "internal/fabricengine/remove.go header and `RemoveStepArchive` comment, and pattern/PATTERN-fabric-git.md background: settle whether they describe the current order, a local archive tag first, local branch deletion, then the child's weft push, tag push and origin delete."
---

The focus entries name areas to settle in round 2 from the round 1 findings; they carry no verdict.
