```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
- **A missing absolute path in push/fetch/pull/ls-remote stderr counts as transport:** `transport_test.go` row "push to a missing local path", plus real git push, fetch and ls-remote in `TestIsTransportFailure_RealGit` (`transport_integration_test.go:60-76`). Pull has only the hand-built form. Pinned.
- **The clone form `repository '&lt;path&gt;' does not exist`:** row "clone of a missing local path", plus the real clone in `TestIsTransportFailure_RealGit`. Pinned.
- **The `file://` prefix is stripped:** row "ls-remote of a missing file URL". It is a hand-built stderr, so only that row pins the strip.
- **A relative path is read against `GitError.Dir`:** row "fetch of a missing relative path". Only a hand-built row pins it; no real-git row uses a relative URL.
- **An existing directory holding no repository is not transport:** row "push to a directory that holds no repository", plus the real push to the recreated empty `bare`. Pinned.
- **A local subcommand naming a missing path is not transport:** row "local command naming a missing path". Pinned.
- **`os.Lstat` is used rather than `os.Stat`, so a dangling symlink to an absent target counts as existing:** none.
- **An unconfigured or mistyped remote name, which git quotes in the same `'&lt;name&gt;' does not appear to be a git repository` line, is not transport:** none.
- **The push classification records a `transport` failure for a removed local origin:** `TestPushFailureClasses/APushToAnUnreachableOriginWritesATransportRecord/OfABranchNeverPushed/ARemovedLocalOrigin`. Pinned.
- **An `other`-class push to an existing non-repository directory:** the existing `other`-class subtest, with its fixture now `t.TempDir()` (`pushclasses_integration_test.go:761`). Pinned.
- **`ReconcileOrigin` returns `Unreachable` with no error for a missing origin:** `TestReconcileOrigin_Judge` row "an origin missing on disk" with `wantUnreachable`. Pinned.
- **`ReconcileOrigin` returns an error with no retry for a non-repository directory:** the row "an origin that is no repository" and the subtest "An origin that is no repository is an error with no retry". Pinned.
- **Add's `RetryOriginReadInTransit` now retries a missing local origin:** none with a sleep count. `TestAdd_RetriesOriginProbesThatFailInTransit` covers only the refused URL and a non-repository directory; the missing-path case reaches only the unseamed test in finding 2.
- **The `pushanchored` fetch from a non-repository directory is never pushed again:** `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, with the fixture changed. Pinned.
- **`shedtransient.Class` now marks a missing local origin `git-transport`:** none directly; it follows from `IsTransportFailure`, which is pinned.
- **The cleanup verb refuses `--apply` over an origin that is no repository:** `TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`, with the fixture changed. Pinned.

**Findings:**

1. **MEDIUM, design.** `internal/gitexec/transport_test.go:56-68`: no row pins that a missing or mistyped remote name is not a transport failure.
   - For a remote that is not configured, git prints `fatal: 'origin' does not appear to be a git repository`. A mistyped remote such as `orgin` gets the same line. I reproduced both in the handler's probe with real git 2.53.0.
   - The new branch would stat `&lt;Dir&gt;/origin`, find it missing and report transport. No test, hand-built or real, has a bare remote-name row, so the behaviour is unpinned in either direction.
   - Evidence: `transport.go:79-91` (`namesMissingLocalPath`) and the handler's real-git probe output.

2. **MEDIUM, consistency.** `internal/fabricengine/add_rollback_adopt_test.go:269-271`: `TestAdd_GitFailureCarriesGitsOwnReason` still points origin at a path that does not exist.
   - The diff makes that path a transport failure, so Add's `RetryOriginReadInTransit` now retries the probe. The test passes `fabricengine.AddOptions{}` with no `BackoffSleep`, so it sleeps through `originReadBackoffDelays` in real time: 1 s, 2 s and 4 s, at least 7 s per probe.
   - That breaks PATTERN-test-speed's clocks rule and the testing skill's no-real-time rule.
   - The test also now exercises the transport-retry path rather than the refusal its comment describes ("fails for a reason only git can state").
   - Evidence: `add.go:81`, `add.go:89-104`, `add.go:296` and `add_rollback_adopt_test.go:269-275`.

3. **LOW, consistency.** `internal/fabricengine/clone_singlerepo_integration_test.go:775-776,877-878`: the `OriginCannotBeRead` row's field comment says "a path that holds no repository", but the fixture is `filepath.Join(t.TempDir(), "no-such-repository")`, a missing path.
   - The diff makes that path a transport failure. The row now checks a missing origin, which the comment does not say.
   - The other four fixtures that switched to `t.TempDir()` were updated; this one was not.
   - Evidence: `clone_singlerepo_integration_test.go:775-776` and `878`.

4. **LOW, design.** `internal/gitexec/transport.go:89`: no row pins the `os.Lstat` choice.
   - A remote path that is a symlink to an unmounted target counts as existing, so it is a refusal.
   - The goal's "unmounted directory" case can take that shape, and no test states whether it should count.
   - Evidence: `transport.go:89-90`.

5. **LOW, design.** `internal/fabricengine/add_remoteleftover_integration_test.go:102-105`: Add now retries a missing local origin, and no `TestAdd_RetriesOriginProbesThatFailInTransit` row pins it with a sleep count.
   - The `OriginStaysDown` row uses only the refused URL.
   - Evidence: `add_remoteleftover_integration_test.go:102-105` and `add.go:96`.

Tool calls:
1. Read the rows of `add_remoteleftover_integration_test.go:90-112` and grepped the callers of `RetryOriginReadInTransit`. My context held only the diff hunk, not the row table or the Add call sites.
