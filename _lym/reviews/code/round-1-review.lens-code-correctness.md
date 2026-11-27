```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function or branch the diff adds or changes through concrete cases.

**List**

- `fabricSync`, `internal/webstercli/sync.go:25`:
  - nil `open` with `SkipGit` true: returns (false, nil) before anything is opened. Correct, and the order of steps matches the base.
  - `SkipGit` true with a failing opener: the opener is never called, so it returns (false, nil). Correct.
  - Zero options with a failing opener: the open error comes back as (false, err). Correct.
  - Zero options when the commit lands but the correspondence record fails: returns (true, err). Correct; `TestFabricSync_ReportsCommittedWhenCorrespondenceRecordFails` covers it.
  - The options reach `CommitWithTrailers` unchanged, so `SkipPush` is honoured. Correct.
- `masterStrandLookup`, `internal/webstercli/batchpause.go:17`:
  - Name "": the error names `LYM_STRAND_NAME`. Correct.
  - Name "m": `resolve("m")` returns the guid. Correct.
  - Resolve fails: the error is returned as is. Correct.
  - The closure captures the name at wiring time. That matches the per-process claim, since each process wires once.
- `wireHub` and `wireStandalone`, `internal/webstercli/wiring.go:218-220, 357-359`:
  - The three fields are set after every early `return err`.
  - Every error branch of `resolvePersistentPreRun` (`cli.go:208-231`) aborts or replaces the verb, so no verb runs with unwired zero fields.
  - Production values are read from the same variables as before. Correct.
- `refuseOrphanedMaster`, `internal/webstercli/ownership.go:59`:
  - It reads `c.strandName`. For the cases "", a Master strand under a held lock, and a Master strand with the run ended, it gives the same result as the old environment read in a wired process. Correct.
- `reset`, `internal/webstercli/reset.go:141, 186`:
  - `ResetPairCode` and `fabricSync` get `c.syncOptions`. In hub mode that equals the old `EnvSyncOptions()`.
  - Standalone mode takes the keep-reset branch and a nil opener, which is unchanged. Correct.
- Every verb's `fabricSync` call site (`acceptaudit`, `beginbatch`, `fixplan`, `fixverify`, `rebaseline`, `recordbatch`, `recoverbatch`, `restoreplan`, `run`): each passes `c.syncOptions` in the right argument position. The behaviour is correct.
- Test lifetimes:
  - Parallel subtests under a serial parent that calls `t.Setenv`, in the commitstatus, discussion-record, `TestVerifyVerb` and `TestGateParity_VerifyGate` tests: Go runs the parent's cleanups only after its parallel subtests finish. No other top-level test runs while a serial test is running. The `BOARD_SKIP_GIT` and `PATH` settings therefore stay in force for each subtest. Correct.
  - `TestBatchVerbs_RefuseUnderAPause`: its parallel subtests start only after the serial logcapture rows have returned and run their cleanups. Correct.
  - `TestResetCmd`: each subtest builds its own fixture from its own hub copy. Loop variables are per-iteration under Go 1.26. Correct.
  - `TestCirclingCLI_PreRunResolvesTarget`, newly parallel: `clihelp.ExecuteIn` passes the working directory through the context, with no `Chdir`. `cobra.EnableTraverseRunHooks` is changed only by the serial `TestLoomVerbs_StaleStamp...`, which restores it in cleanup before any parallel test starts. Correct.
  - The commitstatus and discussion rows that now share the slug "pair": each row has its own hub copy, so board entries, filing marks and record files cannot collide. Correct.

**Findings**

None. The new and changed branches I ran all give correct results, and none swallows or misreports an error or leaves state inconsistent on a failure path.

Out of lens:
- Nine production files are not gofmt-clean, `c.syncOptions,"…"` with no space, for example `internal/webstercli/acceptaudit.go:108`. Only the `parent-align` and `finalize` gates run the gofmt guard.
- The doc comment at `internal/loomcli/hubfixture_helper_test.go:373` lost a space: "creates&lt;anchorPath&gt;".

**Tool calls**

1. `sync.go` body, the return points in `wiring.go`, and the `resolvePersistentPreRun` branches: the diff showed only hunks, not whether the SkipGit check comes before the opener or whether wiring sets the fields after every early return.
2. `cli.go:196-233`: to confirm that every pre-run error branch aborts, so no verb runs unwired.
3. `fabricengine` Bolt-replay registration and `refuseConfigLoad`: to check whether the template build order could change the `AddPair` result in `TestRunOwnerVerdict`. The search did not settle it.
