---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/remoteteardown.go:295
    summary: spawnTeardownHold, the real detached-child path, and the teardown child verb's RunE have no test at any tier
    fix: "Give spawnTeardownHold an injected child command (an executable path plus its arguments, passed from holdTeardown through a Topology field defaulting to os.Executable and teardownChildArgs) so a test can start a stand-in. Add one tier-1 table test in remoteteardown_childhold_integration_test.go, using the repo's re-exec TestMain pattern for the stand-in, whose rows are a stand-in answering held (a childHold comes back, the lock file it made is recorded in rec, and a record with role RoleTeardown exists under the slug in the owner record), busy with try set (an error wrapping ErrPairBusy), busy without try (an error wrapping ErrPairPushRunning naming the rerun text), and an error line or a closed stdout (an error naming the answer). Keep the existing childHold.finish rows."
  - id: F2
    severity: LOW
    class: design
    location: internal/fabriccli/teardownchild.go:26
    summary: A teardown child ended by a signal before its last step, and a job the parent cannot write to it, reach the fabric log only and mail no creel notice
    fix: "Export from internal/fabricengine/remoteteardown.go a function that calls postRemoteTeardownNotice with a line naming the slug, the operation (the remote teardown ended before its last step) and the way forward, reuse rerunWayForward's wording for the kinds a signal cannot tell apart (remove, or cleanup, or the discarding command). Call it from warnWhenEnded in internal/fabriccli/teardownchild.go before os.Exit, passing the hub path (pass hubLocation.HubPath into warnWhenEnded). Call it too from childHold.finish when the job write fails, replacing the bare logger.Warn there. Add one row for each to the notice tests in remoteteardown_test.go."
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/remoteteardown.go:205
    summary: The notice for an unreadable job names the removal and the cleanup re-run but not the discard, though a discard job can be the unreadable one
    fix: "In noticeUnreadRemoteTeardown, extend the way forward with a third alternative, or the command that discarded the task branch again when a task-branch discard started it. Extend TestRunTeardownChild_NoticesAnUnreadableJob to assert the discard alternative is present."
  - id: F4
    severity: LOW
    class: design
    location: internal/fabricengine/remoteteardown.go:160
    summary: A failure that is not an archive failure is reported as one with a tag-push way forward, and a failed worktree resolution points the manual tag push at the warp worktree
    fix: "In failedRemoteSteps, when lymcwd.ResolveWorktree(job.Worktree) fails, drop the manual `git -C ... push` tag command and give only the re-run, since no weft repo root can be named. In runRemoteTeardown, return the weft-root resolution failure as its own step (a distinct operation text such as the lookup of its records repository) instead of through out.err, so the way forward does not tell the operator to push a tag that archiveWeftTip already pushed. Add rows for both to TestNoticeRemoteTeardown."
---

### [MEDIUM] spawnTeardownHold and the teardown child verb are untested

**Location:** internal/fabricengine/remoteteardown.go:295, internal/fabriccli/teardownchild.go:40
**Issue:** `holdTeardown` takes the in-process branch whenever `testing.Testing()` is true, so no test in any tier reaches `spawnTeardownHold`. Its behavior is the whole detached handover: the held, busy and unknown answers, the refusal a busy child returns, the lock file recorded in `rec`, and the registration under `runproc.RoleTeardown` that `pairteardown` relies on to spare the child. `teardownChildCmd`'s `RunE` is covered only by the arity and role-guard tables, which never run it. The round-1 fix moved `childHold.finish` to tier 1 and covers the writing side, and `RunTeardownChild` is covered for the lock and the step order. The process boundary between them, which the focus asked about, has no row that would fail if the answer protocol, the registration role or the lock-file record broke.

### [LOW] Two ways the remote steps never run leave no creel notice

**Location:** internal/fabriccli/teardownchild.go:26, internal/fabricengine/remoteteardown.go:276
**Issue:** The decision record sends an async remote failure to the fabric log and to the run's orch as one creel notice. Every failed step that `runRemoteTeardown` reaches does both, through `postRemoteTeardownNotice`, and an unreadable job is mailed too. Two paths end the job without a notice. `warnWhenEnded` logs a Warn and calls `os.Exit(1)` when a signal arrives mid-job, so a deliberate `lym tender end` of the slug stops the steps with a trace in the log and nothing mailed. `childHold.finish` logs a Warn when the job cannot be written, and the child then reads an empty input and logs an Info line. The `EndSparing` question settles as follows: the later teardown of the same slug spares `RoleTeardown` and cannot end the child, and the tender end that can (`internal/tendercli/deps.go` calls `runproc.End` on the slug) is an operator's deliberate act that leaves a Warn naming the way forward. So no end is without a trace, but only the log carries it.

### [LOW] A non-archive failure reads as an archive failure, and a failed worktree resolution misdirects the manual tag push

**Location:** internal/fabricengine/remoteteardown.go:111, internal/fabricengine/remoteteardown.go:117, internal/fabricengine/remoteteardown.go:164
**Issue:** `runRemoteTeardown` stores the `resolve weft repo root` failure, which comes after the archive succeeded, in `out.err`. `failedRemoteSteps` then reports it as the failed archive of its records with the way forward "push the archive tag", though the tag is already on origin. When `RunTeardownChild` cannot resolve `job.Worktree`, the same branch falls back to `job.Worktree` as the weft root, so the printed `git -C <warp worktree> push origin refs/tags/...` names a repository that holds no such tag.

### [LOW] The unreadable-job notice omits the discard way forward

**Location:** internal/fabricengine/remoteteardown.go:205
**Issue:** `noticeUnreadRemoteTeardown` cannot know the kind of a job it failed to read, yet its way forward names only `lym fabric remove <slug> --remote` and the cleanup sweep. A discard job can be the unreadable one, and for it neither named re-run is the command that started the teardown.

## Focus verdicts

- **Remote steps reaching log and notice (`noticeRemoteTeardown`, `deleteTaskBranchOnOrigin`).**
  The archive-tag push, the leased weft delete and the task-branch lookup, fetch and delete each reach the Warn log and one creel notice naming slug, operation and way forward.
  The weft-tip push refusal is logged and not mailed, which `PATTERN-push-both-sides` states for a teardown's weft push, and the archive tag still holds the tip, so that is not a defect.
  A job reports every step that ran, the weft delete and the task-branch delete in one notice.
  A failed archive stops the later steps by design, so only it is reported, and its way forward is the re-run that retries them all.
- **Pause exemption and docs.**
  The job carries `GateExemptsAll` of the command that wrote it, so a removal's child runs exempt as the command does, and a gated cleanup or discard child keeps its gate.
  `internal/fabriccli/doc.go` states the removal's case, and the lock refusal text now names the stuck teardown child and the pause wait.
  No inconsistency.
- **Test rows.**
  `RunTeardownChild` running a removal job is pinned for the weft tip, the tag, the leased delete, the lock held at each step and its release, and a failed archive stopping the later steps.
  `childHold.finish` and the envelope's `remote_handed_off` have rows, and `noticeRemoteTeardown` has rows per job kind.
  `spawnTeardownHold` has none, which is F1.
- **`EndSparing` and the teardown child.**
  See F2 for the settled answer.
- **Default way forward and unreadable job.**
  The default way forward follows the job kind, with the cleanup sweep for a cleanup job and the slug's removal otherwise, and an unreadable job is mailed.
  The unreadable job's text omits the discard, which is F3.
- **Comments and background.**
  The `remove.go` header, the `RemoveStepArchive` comment and `pattern/PATTERN-fabric-git.md` describe the current order: the local tag first, the local branch deletion, then the child's weft push, tag push and origin delete.
  The `PATTERN.md` entry for `PATTERN-slug-remote-order` carries the held-by-the-child clause.
