```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each behavior the diff changes, I list what reads or asserts it and whether the diff covers it.

**Per change, the set found**

1. **Local-only mode removal.**
   - A repo-wide grep finds no remaining code or doc reference to `--local`, `lymlocal`, `BranchMarkedLocal`, `MarkBranchLocal`, `ClearBranchLocalMark`, `ErrLocalBranch`, `LocalSkipped`, `local_skipped`, `local_mark*`, `AddLocal`, `ErrLocalAddOverOrdinary` or `dropped_records_tip`. Covered.
   - The only hits are `docs/research/*` dated records and `calls.tsv`, which are history, not findings.
   - `internal/fabricengine/parenttips_integration_test.go:101` uses "local-only" only as a test row name for a branch with no upstream, so it is unrelated.
   - The `fabriccli/doc.go`, `fabric.go` help, `weft_verbs.go` and `SANDBOX-FABRIC-SUITE.md` edits are in the diff.
   - `fabric.go`'s `removeFields` drops `local`, `local_mark_branch` and `dropped_records_tip`.

2. **Teardown child verb `remote-teardown`.**
   - Cobra wiring: `fabric.go` calls `AddCommand(teardownChildCmd())`. Covered.
   - Role guard: `roleguard.go:49` passes the verb, and `roleguard_test.go:38` pins it. Covered.
   - Arity: `argsarity_test.go:38` pins it. Covered.
   - Process registry: `spawnTeardownHold` registers the child under `runproc.RolePush`. `anchor.go:411` and `anchordeps.go:447` spare push and sync roles when ending dobbies, so the child survives a worktree move. Covered.

3. **`RemoveResult.RemoteHandedOff` and the empty remote fields in production.**
   - `removeFields` emits `remote_handed_off`, and the field list comment is updated. Covered.
   - Consumers of the now-empty outcome fields: `battencli/wire.go:379` (`RemoteBranchError`), `fabriccli/fabric.go:1119-1151` (`RemoteBranchError`, `RemoteBranchDeleted`, `RemoteCodeBranchDeleted`) and `fabricengine/weftwiring.go:236` (`entry.RemoteDeleted`). I did not read whether each handles `RemoteHandedOff`. See findings 2 and 3.

4. **`DiscardTaskBranch` and `Cleanup` going async.**
   - `battencli/cancel.go:682` copies `discard.ArchiveTags`, and `cancelcmd.go:161` reports it. `DiscardResult` has no handed-off marker, and `ArchiveTags` falls back to the local tag. Neither file is in the diff. Not confirmed covered.
   - `CleanupBranchEntry` and `CleanupResult` have no handed-off marker either. See finding 3.

5. **`PATTERN.md` and `pattern/`.**
   - The `PATTERN-slug-remote-order` text is amended in the diff, and the handoff clause matches the code. No `pattern/` background file exists for that entry.
   - `pattern/PATTERN-fabric-git.md` (lines 30-36) stays accurate.
   - No declared-changes list was rendered into my context, so I raise nothing under "Declared PATTERN changes landed".

6. **Commit messages.** None of the five commits carries a `Pattern-Departure:` trailer, and I found nothing that needs one.

**Findings**

1. **MEDIUM, design:** the removal's pause-gate exemption does not reach the remote steps that now run in the child.
   - Location: `internal/fabriccli/remove.go:42`, `internal/fabriccli/doc.go:58`, `internal/fabriccli/teardownchild.go:35-47`.
   - Evidence: `doc.go` says the removal exempts its own network gate from every live pause so its network calls pass, and that "its detached children keep their own gates". The weft-tip push, archive-tag push and leased delete now run in the child, which never calls `pauserecord.SetGateExempt`. Under a pause, the child can hold the exclusive teardown lock while it waits on the gate. The docs name the cost (the refusal text tells the operator to end a stuck teardown child). They do not say that remove's own remote steps moved from exempt to gated.
   - Fix direction: judge whether the child should take the same exemption, or document the new behavior.

2. **MEDIUM, design:** consumers of the remote outcome fields are not shown to handle the handed-off case.
   - Location: `internal/battencli/wire.go:379`, `internal/fabriccli/fabric.go:1119-1151`.
   - Evidence: in production `Remove` returns with `RemoteBranchError`, `RemoteBranchDeleted` and `RemoteCodeBranchDeleted` zero, and only `RemoteHandedOff` set. `battencli/wire.go:379` reads `RemoteBranchError` and treats empty as success. `fabric.go:1119` and `fabric.go:1145-1151` still publish `remote_branch_deleted: false` and similar keys beside `remote_handed_off`. A caller reading `remote_branch_deleted: false` can take that as "not deleted" rather than "pending". The diff does not touch `battencli/wire.go`.
   - I did not read lines 1119-1126 in full to see whether a handed-off guard exists, so this needs checking against the code.

3. **LOW, consistency:** `Cleanup` and `DiscardTaskBranch` hand off with no signal in their results.
   - Location: `internal/fabricengine/cleanup.go:320-327`, `internal/fabricengine/taskbranchdiscard.go:66-71`, `internal/fabricengine/weftwiring.go:236`.
   - Evidence: when the job is handed to the child, `out.ran` is false, so `RemoteDeleted`, `OriginDeleted` and the remote archive tags stay zero. Unlike `RemoveResult.RemoteHandedOff`, `CleanupBranchEntry`, `CleanupResult` and `DiscardResult` carry no marker. `weftwiring.go:236` reads `entry.RemoteDeleted` and gets false in production. The result and the CLI envelope cannot tell "origin delete pending" from "origin delete not done".

**Tool calls (3)**
- Grep for leftover local-only symbols, `remote-teardown` references, doc phrases and `PATTERN-slug-remote-order`: the diff lists files but not what lies outside them.
- Read `fabriccli/doc.go` lines 50-64 and grep for the pause exemption and `RolePush` consumers: to check whether the child inherits the gate exemption and how the role is used.
- Grep for non-test readers of the remote outcome fields: to find consumers outside the diff.
