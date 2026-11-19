```yaml
lens: code-affected-set
outcome: reported
```

The lens list is below: per change, the set I found and whether the diff covers it. The three findings that follow come out of that list. I made 3 tool calls.

**Per change, the set found**

1. **`oneRepositoryAt` / `recordsBranchIn` / `recordsBranchAt`: covered inside fabric, not outside.**
   - Most fabric callers moved to `recordsBranchIn`.
   - `RecordsBranchName(` is still called in `internal/reportcli/cli.go:205`, `internal/battencli/readoptstop.go:224` and `internal/reportengine/timing.go:363`.
   - `internal/fabricengine/doc.go` says callers outside fabric keep the two-repository naming.
   - D4 names `reportcli` and `battencli`'s catch-up as readers that must read "the same repository unchanged".
2. **`MidMergeRecordsUnsettled` (new `MidMergeKind`): covered.** The consumers outside `midmerge.go` are `landingshed/finalize.go:581`, `landingshed/premerge.go:51`, `loomcli/start.go:181` and `loomcli/start.go:238`, and all four were updated.
3. **`PushFailureOperatorUnpushed` / `PushFailureProtected` (new failed-push classes): covered.** The class switches are all in fabric (`failedpush.go`, `pushanchored.go`) and were updated. The `doc.go` text was updated and no other docs list the classes.
4. **`runfsm.Inputs.PushOwedWayForward`: partly covered.**
   - The only setter, `pairteardown/runread.go:239`, was updated.
   - The `aside.PushOwed` reset in `Judge` (`runfsm/act.go:258-262`) was not extended.
   - `pattern/PATTERN-run-fsm.md` still says the refusal names `lym fabric push` in the owed worktree.
5. **`statuscommit.Deps` (new fields): covered.** Both wrappers, `loomcli` and `battencli`, were updated, and so were their comments, `statuscommit` doc and `docs/overview.md`.
6. **`MergeStateActive` / `MergeStateOperation`: covered.** Callers and the `battencli`/`loomcli` dep comments were updated.
7. **`gitrepo.MergeStart` → `mergeStart` and `MergeStartNoFastForward`: covered.** The pinned run-bound list in `cmd/lym/gitrepoboundary_test.go` and `gitrepo/doc.go` were updated.
8. **Allowlists in `cmd/lym` (destructive and uncontained write): covered for the new files that destroy or write.** I did not confirm that `junction.go`'s new `os.MkdirAll` in `wireOneRepositoryDirs` is already allowlisted; that file's existing entry is outside my context.
9. **Declared PATTERN changes: covered.**
   - `PATTERN.md` carries the `PATTERN-single-repo-hub` line, the `PATTERN-run-fsm` line and the `PATTERN-cwd-resolution` line with the declared text.
   - The background paragraphs the record's Constraints list landed for `fabric-boundary`, `lym-files`, `cwd-resolution`, `pair-remote` and `run-fsm`, plus the new `single-repo-hub` background.
10. **Docs: `docs/getting-started.md` and `docs/overview.md` covered; `README.md` not** (finding 1).

**Findings**

1. **MEDIUM, class `design`: `README.md` lacks the clone description and the hazards D2 requires.**
   - Location: `README.md` (the diff changes two table cells only).
   - D2 and the record's acceptance bullet "Review: D2" say `docs/getting-started.md` and `README.md` both describe cloning a one-repository hub, what it puts on code branches, the `--single-repo` hazard on a two-repository hub's code repository, and the switch/reset/stash hazards.
   - The README diff names the layout and `lym-bolt` in the run table and the glossary only. `grep` finds no `single-repo`, `lym.primeBranch` or `git stash` in `README.md`.
   - Plan card 15 (`_lym/plan/15-one-repository-docs.md`) narrows the README to those two table cells, so the card departs from the decision and the diff follows the card.
   - Evidence: `README.md` diff hunks at lines 36 and 132; `docs/getting-started.md` diff lines 9-31 carry the full text.

2. **MEDIUM, class `design`: reader-side records-branch derivation outside fabric is not layout-aware.**
   - Locations: `internal/reportcli/cli.go:205` (`RecordsBranch: fabricengine.RecordsBranchName(parentBranch)`), `internal/battencli/readoptstop.go:224` (`recordsBranch := fabricengine.RecordsBranchName(taskBranch)`, then `BranchFile(recordsBranch, …)` at `:253`), and `internal/reportengine/timing.go:363`.
   - In a one-repository hub the records branch is the code branch itself.
   - These weft-side readers outside fabric (`reportcli`, `battencli`'s catch-up, `reportengine`) would look for `&lt;branch&gt;-weft`, which does not exist. D4 says they read "the same repository unchanged".
   - Fabric's own doc states the outside callers keep the two-repository naming, but none of these three was changed or guarded.
   - Evidence: the `grep` of `RecordsBranchName(` above, and D4's bullet on `RecordsWorktree`/`RecordsRepoRoot`.
   - I did not trace the first two, `reportcli` and `battencli` catch-up, to the end of a one-repository run. The `reportengine/timing.go:363` use is a suffix strip and may harmlessly no-op.

3. **LOW, class `consistency`: the new `PushOwedWayForward` is not cleared where its sibling owed fields are, and the pattern background is stale.**
   - In `Judge`, the `aside.PushOwed` block (`internal/runfsm/act.go:258-262`) resets `PushOwed`, `PushOwedError` and `PushOwedWorktree` but not `PushOwedWayForward`. A reset with a stale way forward would only matter if a later change reads the field with `PushOwed` false; today `Refusal` only reads it under `ReasonPushOwed`, so this is a latent inconsistency.
   - `pattern/PATTERN-run-fsm.md` (changed in this diff) still reads "The act rule's refusal names `lym fabric push` in the owed worktree", though the refusal now names the record's own way forward, such as `git push`, for `operator_unpushed`.
   - Evidence: `act.go:258-262`, and the `pattern/PATTERN-run-fsm.md` hunk at diff lines 84-90.

**Tool calls**
1. `grep` for `MidMergeParked`/`MidMergeForeign`, `teardown_lock`/`PushFailureTeardownLock`, `RecordsBranchName(` and `PushOwedWorktree` across Go and md files: callers beyond the diff were not in context.
2. Read `_lym/plan/15-one-repository-docs.md` and `grep` of `README.md` for clone/hazard terms: to see what the card asked of the README.
3. Read `internal/runfsm/act.go` lines 240-275 and `grep` for `recordsBranch` in `reportcli`/`battencli`: the `Judge` reset block and the readers' use were not in context.
