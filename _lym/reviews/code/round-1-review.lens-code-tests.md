```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. **`ruleOutsideCreated` from `gitInvocationMutates`.** Pinned by the `TestMutatingGitCommandsNameTheBrokenRule` row "commit outside every created repository" (`cluster_test.go:750`) and by the review-path row at `cluster_test.go:309`, which asserts the full `counted as …` substring.
2. **`rulePushAfterLink` via `redirectPushes` from `runsMutatingGit`.** Pinned by the "push after ln" row (`cluster_test.go:751`); without the ln rule the self-push passes and the test stops on `Fatal`. The incident row (`cluster_test.go:313-326`) pins the same rule end to end.
3. **`redirectPushes` keeps the first rule across calls.** Pinned by the "push after a config key and a later ln" row (`cluster_test.go:752`).
4. **`rulePushAfterUnknownRemoteChange` from `recordRemote`.** Pinned by the row at `cluster_test.go:753`.
5. **`pushMutates` rules:**
   - `rulePushWithRepo`: row at `cluster_test.go:754`.
   - `rulePushSetsUpstreamOutside`: row at `cluster_test.go:755`.
   - `rulePushNoDestination`: row at `cluster_test.go:756`.
   - `rulePushToRemoteOutside`, bare word never pointed: row at `cluster_test.go:757`.
   - `rulePushToRemoteOutside`, named remote pointed outside: row at `cluster_test.go:758`.
   - `rulePushToPathOutside`, path: row at `cluster_test.go:759`.
   - `rulePushToPathOutside`, URL: row at `cluster_test.go:760`.
   - The `""` returns for a remote that leads in and a path that leads in are still pinned by the "" rows of `TestFirstMutatingGitCommandScopesToForkRepositories`, such as `cluster_test.go:614-615`.
6. **`runsMutatingGit` takes the rule of the command's first counted invocation (`if rule == ""`).** None. Every row in the rule test has at most one counted invocation per command. The one multi-counted command (`cluster_test.go:134`) asserts only "git-mutating command". This is F1.
7. **One call that both names a redirect key and changes a remote (a focus question).** None. It resolves through the first-wins rule of item 3, and I have not filed it separately.
8. **`forkViolation` message names `found[0].rule`.** Pinned by `cluster_test.go:309` and `cluster_test.go:324`.
9. **`auditClusterRound` appends `violationWayForward`:**
   - Revert branch: row at `cluster_test.go:310`.
   - Untouched-with-push branch: the ln row at `cluster_test.go:325`.
   - Untouched-without-push branch on the review path: none. The review-path "named spawns" row (`cluster_test.go:327-336`) sets no `wantWayForward`. The same branch is pinned on the fix path at `cluster_test.go:229`, and both paths call the one function, so this is no gap.
10. **Rename `fixViolationWayForward` → `violationWayForward`.** The fix-path rows still pin it; the compiler catches the rename.
11. **The way-forward assertion moved out of the `fix` branch and now runs when `wantWayForward != ""`.** Not loosened: every fix-path row with `wantErr: true` sets `wantWayForward` (`cluster_test.go:107`, `118`, `129`, `140`, `151`, `190`, `205`, `229`). At the base, that suffix check applied only to fix rows with `wantErr`, so no row is checked more loosely now. The `ErrClusterForksMissing` rows still assert `errors.Is` (`cluster_test.go:378-380`).

**Findings**

- **F1 — LOW — design**
  - Location: `internal/burlerengine/cluster_test.go#TestMutatingGitCommandsNameTheBrokenRule`
  - Summary: no row pins that a command with several counted invocations names its first one's rule. If `runsMutatingGit` kept the last rule instead of the first, every test would still pass.
  - Evidence: `cluster.go:235-237` sets the first rule. The doc comment at `cluster.go:141` and the struct field comment at `cluster.go:132` both state that the first counted invocation's rule is named. Every row at `cluster_test.go:750-760` has at most one counted invocation per command, and `cluster_test.go:138` asserts only "git-mutating command".
  - Fix: add a row to `TestMutatingGitCommandsNameTheBrokenRule`, such as `{"first counted invocation names the rule", []string{"git commit -am x && git init /tmp/r && git -C /tmp/r push"}, ruleOutsideCreated}`. Its second counted invocation would return `rulePushNoDestination`.

- **F2 — LOW — consistency**
  - Location: `_lym/landing/summary.md`, Tests section
  - Summary: the change description claims "Every row's way forward is checked on both the review and the fix paths". Each row runs on one path only, through `if tt.fix`, and the review-path rows for nested spawns, writes and named spawns set no `wantWayForward`.
  - Evidence: `cluster_test.go:355-366` and the rows at `cluster_test.go:276-298` and `327-336`.
  - Fix: reword the claim to "the way-forward check now applies to review-path rows as well as fix-path rows, and the review-path commit and ln rows assert it". Alternatively, add `wantWayForward` to the review-path nested-spawn, write and named-spawn rows.

Tool calls: none; my inherited context held everything this lens needed.
