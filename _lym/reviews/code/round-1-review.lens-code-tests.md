```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff adds or changes, the test that pins it.

**Per-behaviour list**

1. **Hook splits the worktree root with `${WORKTREE_ROOT%/*}` and `${WORKTREE_ROOT##*/}`** (`post-checkout.sh:35-36`).
   - Pinned by `TestInstallPostCheckoutHook_WeftResolution_Prime` (`hook_test.go:170`) and `_Child` (`hook_test.go:225`).
   - Both do real checkouts. If the weft sibling path were wrong, `[ ! -d ]` would exit silently, and the diverged case's `fabric:` assertion (`hook_test.go:210`, `:274`) would fail.
   - Both the warn-on-mismatch and silent-on-match paths are covered, so the acceptance criterion holds.
   - No root-level worktree such as `/x` is covered. That case is outside the type system's guarantees but outside any realistic test, so I raise no finding.

2. **Chain wrapper finds its directory with `case "$0"` and `${0%/*}`** (`hook.go:44-47`).
   - Pinned by the new checkout probe in `TestInstallPostCheckoutHook_ChainIdempotent` (`hook_test.go:152-161`).
   - Git invokes the hook with a path containing a slash, so the `*/*` branch fires. If `SCRIPT_DIR` were wrong, the user hook would not run and `"user\n"` would be missing from the output.
   - The `*) SCRIPT_DIR=.` fallback branch has no test. A `$0` with no slash needs a direct `sh post-checkout` invocation, which git never does. Its exercise by a test is impractical.

3. **Wrapper runs `post-checkout.user` when it is executable** (acceptance criterion).
   - Pinned by the probe above (`hook_test.go:152-161`).
   - The probe's assertion is `strings.Contains(out, "user\n")`. The user hook echoes `user`.
   - One weakness: the git checkout output is stderr and stdout combined. The check cannot be fooled by the fabric hook's own output, because that output starts with `fabric:`. It is adequate.

4. **Wrapper skips a non-executable user hook** (unchanged behaviour).
   - Pinned only statically, by a string match on the `-x` guard in `TestInstallPostCheckoutHook_ChainedWrapperIsExecutable` (`hook_test.go:336`).
   - The diff leaves that line unchanged, and the probe is the only runtime test of the wrapper. Out of the diff's change, so no finding.

5. **Chain idempotency, with the new preamble content** (`hook.go:142`).
   - Pinned by `TestInstallPostCheckoutHook_ChainIdempotent` (`hook_test.go:143-150`).
   - A second install compares the content and finds it unchanged.

6. **`newRecordsFixture` takes `CopyHub`, and with it the `EndedSet` rows** (`replay_integration_test.go:39`).
   - The `EndedSet` rows (`:459-547`) call `newRecordsFixture` at `:467`, `:484`, `:502`, `:520` and `:538`, so they take `CopyHub` through the one fixture.
   - Pinned by `TestRecordsReplay` itself. It is a fixture swap, not a behaviour change.

7. **The adoptcli sites take `CopyHub`** (`adopt_integration_test.go:58`, `:216`, `:239`; `check_integration_test.go:30`; `deploy_integration_test.go:78`).
   - Pinned by the same tests, which still run unchanged.
   - Grep shows no `NewHub` left in `internal/adoptcli`.
   - Isolation: `deploy_integration_test.go` is parallel and takes one copy per subtest. It is safe.

**Findings**

None. The probe is not redundant: `_Prime` and `_Child` never install a chained wrapper, because no pre-existing hook exists in their hubs, so none of them runs the chained user hook.

**Tool calls**
1. Read `hook_test.go` lines 160-359, to confirm the `_Prime`, `_Child` and `ChainedWrapperIsExecutable` coverage; the manifest held only lines 90-175.
