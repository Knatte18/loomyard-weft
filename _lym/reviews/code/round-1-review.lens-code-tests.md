```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests, one entry per behaviour the diff adds or changes, naming the test that pins it.

**Per behaviour**

1. `commitWeftAt` seeds the weft excludes before staging (D2).
   - Pinned by `TestBolt_DirtyRepo_CommitsAndPushes`, `internal/fabricengine/bolt_integration_test.go:70-94`.
   - It writes `.lym/logs/trace.log` into an unseeded fixture and asserts `LsFiles == ["bolt-note.md"]`.
   - Reverting the seed makes the trace log tracked, so the assertion fails.
   - Not duplicated. `weftgit_exclude_test.go:157` covers the seed through `Fabric.Commit`, not Bolt's stage-all. `commitweftat_test.go` has no `.lym` assertion.

2. A seed error returns bare from `commitWeftAt` and fails the commit (D2 "A seed failure fails the commit").
   - Pinned by none.
   - The diff adds the branch at `recordsgit.go:376-378`, and no test makes the seed fail.
   - This is a behaviour the diff adds with no coverage.
   - The error already carries the `fabricengine: seed weft artifact excludes in %s` prefix, so the bare return is not a wrapping defect.

3. The seed is idempotent across repeated `Commit` calls, and the `SkipGit` path does not seed.
   - Idempotence is pinned by `weftgit_exclude_test.go:157-170`, a byte-identical exclude after a second `Fabric.Commit`. That path reaches `ensureWeftLockDirAt`, not `commitWeftAt`'s own seed call.
   - `TestBolt_CleanRepo_CommitAndPushAreNoOps` (`bolt_integration_test.go`, around lines 120-140) now calls Bolt's `Commit` after a `Push` on an unseeded fixture. It would fail if the second seed produced a diff, so it indirectly pins the re-seed.
   - The `SkipGit` case (`commitweftat_test.go:107-124`) leaves a dirty file untouched. It does not assert that nothing was seeded, and that behaviour is unchanged by the diff.

4. Dropping the push-lock removal in `TestBolt_CleanRepo_CommitAndPushAreNoOps` is covered by the seeded `.gitrepo-push.lock` exclude.
   - Pinned by that test itself.
   - It asserts `committed == false` after `PushCoalesced` leaves its lock file, and reverting the seed makes the lock get committed and the assertion fail.
   - The `LsFiles` assertion in the dirty-repo test checks tracked files after a commit. The clean-repo test checks that nothing is committed. They cover different outcomes, so there is no duplicate.
   - Open point: `gitkit.LsFiles` compares an exact list. If the fixture's initial commit tracks other files, the want list is wrong. The fixture is `newBoltRepo`, which I did not re-read, so the manifest cannot confirm this either way.

5. The bypass-push step runs from the hub's board worktree through `RunCLIIn(h.BoardDir(), …)` (D1).
   - Pinned by `TestRunCLI_HubMutationScenario/BypassPushAdvancesBothUpstreams`, `hubmutation_integration_test.go:57-73`.
   - It asserts exit code 0, an OK envelope, and both bare upstream SHAs.

6. After the bypass push, the trace sink points at the test hub's logs dir (`logger.TraceDir() == fabricengine.HubLogsDir(h.Path)`).
   - Pinned by the new assertion at `hubmutation_integration_test.go:70`.
   - It checks the scenario's goal, since the leak is the sink resolving to the live hub, so it does not just restate the setup.
   - The only other `TraceDir()` tests are `logger/sink_test.go:458,491`, which cover the logger's own resolution and not this redirect. No duplicate.
   - `logger.SetDurableSinkDir` is process-global, so the scenario cannot run under `t.Parallel` against other tests in the same binary that read the sink.
   - The `t.Cleanup(func() { logger.SetDurableSinkDir("") })` registered in the step's `run` runs when the step's `t` finishes.
   - `TestRunCLI_HubMutationScenario` is parallel as a whole, while the steps are serial per the comment at line 31, so the reset runs before the next step.
   - Other parallel tests in the `fabriccli` binary could still observe the redirected sink between the step and its cleanup. That is a residual risk and no finding.

7. The push-lock file removal in the clean-repo test was dropped. No separate behaviour, covered under 4.

**Findings**

- MEDIUM, class `design` (a coverage gap): the seed-failure path of `commitWeftAt` (`internal/fabricengine/recordsgit.go:376-378`) has no test. D2 states the fail-the-commit behaviour explicitly, and the diff adds the branch. Evidence: no test in `commitweftat_test.go` or `bolt_integration_test.go` induces a seed failure. A row in `commitweftat_test.go`, for example an unwritable `info/exclude`, would pin it. A judgement call: the guidance on internal misuse may argue against a test, but the record names it as a decision.
- No redundancy findings: no new test duplicates an existing one, and the diff adds no per-helper or stencil-prose assertions.

**Tool calls (2 of 6)**
1. Grepped for `.lym`, `ExcludeLines` and `LsFiles` in the three fabricengine test files, and for `TraceDir()` in tests. The manifest did not show whether those tests already covered the exclude seeding or the trace redirect.
