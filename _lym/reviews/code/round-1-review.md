---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remoteteardown.go:150
    summary: "A failed origin task-branch delete (ls-remote, fetch or delete failure) is only an Info log and never reaches the fabric Warn log or the creel notice, and a second failure after a weft-delete failure is dropped"
    fix: "In noticeRemoteTeardown (remoteteardown.go:130-165) add a case after the out.weft.RemoteError case for a failed task-branch delete, and report each failed step instead of only the first. deleteTaskBranchOnOrigin (remove.go:593-609) returns lookup, fetch and delete failures as its kept reason, so split a failure from a gate refusal or lease loss there: return the failure as an error or an extra result field that runRemoteTeardown stores in remoteOutcome, keep a refusal or a moved tip a kept reason that mails nothing. Then build the notice line from every failed step, naming the slug, the operation (the delete of its task branch on origin) and a way forward, and log it at Warn like the other failures."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/remoteteardown_test.go:18
    summary: "The detached handover has no test: RunTeardownChild running a job, spawnTeardownHold, childHold.finish and RemoteHandedOff are unpinned, so the step order and the lock held to the last step are not covered at all"
    fix: "Add an integration-tier test beside the existing remove tests that feeds RunTeardownChild a JSON remoteTeardownJob of kind remove through its jobIn reader on a hubforge hub whose origin lacks the weft tip, and asserts on the bare origin that the weft branch tip, then the archive tag, then the weft branch delete landed in that order (record the order with archiveTagPushHook and removeBeforeWeftDeleteHook), that a TryAcquireReadLock on TeardownLockPath fails until RunTeardownChild returns, and that it succeeds after. Add one row for a job whose archive step fails, asserting the later steps did not run and the lock was released. Add a row on childHold.finish through a fake io.WriteCloser and a started exec.Cmd that pins the nil-job path (stdin closed, Wait returned) and the handed-off outcome. Pin RemoteHandedOff and the remote_handed_off envelope field by one row in removefields_test.go for a result with RemoteHandedOff set."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/remoteteardown.go:130
    summary: noticeRemoteTeardown has no test row, so the log line and the one-line creel notice naming slug, operation and way forward are unpinned for every failure branch
    fix: "Add a table-driven test in remoteteardown_test.go that installs a recording push notifier through the install seam in failedpush.go:84-90 and calls noticeRemoteTeardown with a remoteOutcome per branch: a remove archive failure with and without WeftTip, a discard failure with and without TaskTip, a weft-delete RemoteError, a task-branch delete failure once F1 is fixed, and a clean outcome. Assert exactly one notice per failing row whose text holds the slug, the operation and the way forward, none for the clean row and for a kept branch, and that a nil notifier does not panic."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/fabriccli/teardownchild.go:35
    summary: The removal exempts its own process from every live pause, but the remote steps now run in the teardown child, which installs the network gate without that exemption and can hold the exclusive teardown lock while it waits on a pause
    fix: "Decide and apply one of two changes. Either make the child exempt as its parent was: in teardownChildCmd (teardownchild.go:28-52) call pauserecord.SetGateExemptAll and defer the restore before RunTeardownChild, since the child exists only because an explicit operator verb handed it a job that the verb had already exempted, and update the paragraph at internal/fabriccli/doc.go:58 to say the teardown child shares the removal's exemption while the push children keep their own gates. Or keep the child gated and state in that paragraph and in the refusal text of teardownLockRefusal that a removal's remote steps wait under a pause with the lock held."
  - id: F5
    severity: LOW
    class: design
    origin: handler
    location: internal/fabricengine/remoteteardown.go:265
    summary: The teardown child is registered under the slug with the push role, so a later teardown or tender end of the same slug ends the running child mid-job with no notice, leaving its remote steps undone
    fix: "In spawnTeardownHold register the child under a role the owner-record ends only deliberately, or make the end path of the slug (runproc.EndSparing in internal/pairteardown/teardown.go:621 and the tender End calls) spare the teardown child by a role named in its spare list, and log a Warn from the child when stdin or a signal ends it before its last step, so a killed job leaves a trace with its way forward."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remoteteardown.go:131
    summary: "The default way forward names `lym fabric remove <slug> --remote` for every kind, which is wrong for a cleanup job and returns pair not found for a remove whose origin branches were never pushed"
    fix: "In noticeRemoteTeardown choose the default way forward by job.Kind: for remoteTeardownCleanup name `lym fabric cleanup --apply --remote`, and for remoteTeardownRemove name the re-run only when a branch of the pair can still be read on origin, else name the git push of the archive tag alone."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remoteteardown.go:317
    summary: A job that cannot be read or unmarshalled drops every remote step with only a Warn from the verb, and nothing is mailed or names a way forward
    fix: "In RunTeardownChild, when io.ReadAll or json.Unmarshal fails, call noticeRemoteTeardown (or a sibling that needs no job) with a line naming the slug, the operation (the remote steps of its teardown) and the way forward, as the resolve-worktree failure at remoteteardown.go:321-325 already does."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/fabricengine/remove.go:49
    summary: RemoveStepArchive is documented as the archive tag pushed to origin and as appended only after it changed state, but Remove now appends it when the tag is only made locally and the push is the child's
    fix: "Reword the RemoveStepArchive comment (remove.go:50) to the local archive tag made before the teardown, whose push the teardown child runs, and the doc comment of the Steps field and removePair's step list to match."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: pattern/PATTERN-fabric-git.md:30
    summary: "The background states that every teardown first pushes the archive tag and that a rolled-back Add is the only exception, but the teardown now tags locally, deletes the local branches, and only then has its child push the weft tip and the tag"
    fix: "Reword lines 30-33 and 35 of pattern/PATTERN-fabric-git.md so a teardown tags the local weft tip first and its detached child then pushes the weft branch when origin lacks the tip, pushes the tag, and only then deletes on origin; and update the opening lines of internal/fabricengine/remove.go (lines 1-4, which still say a refused call never pushes a tag and the tag precedes the launcher teardown) to the same order."
---

### [MEDIUM] A failed origin task-branch delete is never reported

**Location:** `internal/fabricengine/remoteteardown.go:150`, `internal/fabricengine/remove.go:593`
**Issue:** `deleteTaskBranchOnOrigin` turns `ls-remote`, fetch and delete failures into a kept reason, which `runRemoteTeardown` stores in `out.taskKept`. `noticeRemoteTeardown` has no case for it, so the `default` branch logs `Info` and mails nothing. The decision asks that every async remote failure reach the fabric log and one creel notice line. The `switch` also reports only the first failing step.

### [MEDIUM] The detached handover has no test

**Location:** `internal/fabricengine/remoteteardown_test.go:18`
**Issue:** Every path that reaches `holdTeardown` runs in process under `testing.Testing()`, and the only `RunTeardownChild` tests feed an empty job or a held lock. `spawnTeardownHold`, `childHold.finish`, the child running a job, the order weft-tip push, archive tag, leased delete, the lock held to the last step and `RemoteHandedOff` have no row that would fail if broken. The focus names the order and the hold, and they are the core of the change.

### [MEDIUM] noticeRemoteTeardown has no test

**Location:** `internal/fabricengine/remoteteardown.go:130`
**Issue:** No test references `noticeRemoteTeardown` or the installed push notifier. The notice line the decision specifies, with the slug, the operation and the way forward, and its failure branches, are unpinned.

### [MEDIUM] The removal's pause exemption does not reach its remote steps

**Location:** `internal/fabriccli/teardownchild.go:35`
**Issue:** `fabric.go:1103` exempts the removal's own process from every live pause, and `doc.go:58` says its detached children keep their own gates. The weft-tip push, the archive-tag push and the leased delete now run in the child, and `main.go:154` installs the gate there with no exemption. Under a pause the child can wait on the gate while it holds the exclusive teardown lock. The summary and the docs do not say the removal's remote steps moved from exempt to gated.

### [LOW] The teardown child is registered as a push under the slug

**Location:** `internal/fabricengine/remoteteardown.go:265`
**Issue:** `spawnTeardownHold` registers the child with `runproc.RolePush` under the slug. `runproc.EndSparing` in `internal/pairteardown/teardown.go:621` ends every process registered under the slug, so a later teardown of the same slug, such as the re-run a notice names, ends a running child mid-job. The end is silent, and the local branches and worktrees are already gone.

### [LOW] The default way forward names the wrong command for a cleanup

**Location:** `internal/fabricengine/remoteteardown.go:131`
**Issue:** The default way forward is a re-run of `lym fabric remove <slug> --remote` for every kind. A cleanup job came from `lym fabric cleanup --apply --remote`, and a remove whose origin branches were never pushed gets `ErrPairNotFound` on the re-run, since `refuseWhenNothingRemains` finds nothing locally or on origin.

### [LOW] An unreadable job is dropped without a notice

**Location:** `internal/fabricengine/remoteteardown.go:317`
**Issue:** A job that fails `io.ReadAll` or `json.Unmarshal` returns an error the verb logs at `Warn`. The remote steps never run, and no notice names the slug or a way forward, unlike the resolve-worktree failure a few lines below.

### [LOW] RemoveStepArchive describes a push that the child makes later

**Location:** `internal/fabricengine/remove.go:49`
**Issue:** The constant is documented as the archive tag pushed to origin, appended only after it changed state. `Remove` appends it once the tag is made locally, and the push is the child's, which may fail after `Remove` returned.

### [LOW] PATTERN-fabric-git background and remove.go header describe the old order

**Location:** `pattern/PATTERN-fabric-git.md:30`
**Issue:** The background says every teardown first pushes the archive tag and that only a rolled-back `Add` is excepted. The teardown now tags locally and deletes the local branches before its child pushes the tag, and `remove.go`'s opening lines still say a refused call pushes no tag and the tag comes before the launcher teardown.

## Rejected

- Lens code-affected-set finding 1, consumers of the empty remote outcome fields (`battencli/wire.go:379`, `fabric.go:1119`): `finishRemoval`'s comment and `removeFields` state the in-process-only semantics, the envelope carries `remote_handed_off`, and the summary and the decision accept that the child mails the failure instead.
- Lens code-affected-set finding 3, no hand-off marker on `CleanupBranchEntry` and `DiscardResult`: the cited consumer `weftwiring.go:236` is a separate synchronous path the diff does not change, and no consumer misreads the zero fields.
- Lens code-tests, `PushCommit` and the `source` parameter of `pushPlainWithRetry` have no row: `TestRemove_ArchiveOrder/BranchIsPushedBeforeTheTagThatNamesIt` pushes the weft tip through them after the local branch is gone.
- Lens code-tests, no new row for the parallel `fabric add` probes: the probes' outcomes are covered by the existing add-leftover tests, and parallelism cannot be pinned without a timing dependence.
- Lens code-tests, no row for an early step skipping the later steps: the reworked `TestRemove_UnreachableOriginFailsClosed` in `remove_archive_integration_test.go` covers a failed archive, and the order is held by the single `return` that F2's new rows cover.
- Lens code-tests, the `remove_localmode_integration_test.go` name and its `Remove` rows: `RemoveLocal` stays by the decision, so the name still fits, and a search finds no other test that sets `SetHousekeepHookForTest` or clears the failed-push records of a removed pair, so the rows duplicate nothing.
