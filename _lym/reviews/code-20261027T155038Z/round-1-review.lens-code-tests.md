```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, then apply test economy.

**Per behaviour, the test that pins it**
- `agentname.FileName` is pinned by `TestFileName` (`agentname_test.go`).
- The flag is set on post and cleared on the last read, forward, withdraw or return, and a program receiver gets none. `TestStore_MailFlags` (`flags_test.go`) has rows `PostSetsTheReceiversFlagAndTheLastReadClearsIt`, `ForwardMovesTheFlag`, `WithdrawClearsTheFlag` and `ProgramReceiverGetsNoFlag`. No row covers return, the fourth path the acceptance criterion names. A journaled change that happens not to alter the set of unread strands is not tested either.
- A daemon start reconciles a stale or missing flag. `ReconcileRestoresALostFlag` and `ReconcileRemovesAStaleFlag` call `store.ReconcileFlags()` directly. None of the tests that call `Run` or the daemon rig asserts the call in `server.go` (the `Run` hunk). Deleting that call breaks nothing.
- The flag-sync-on-failure branch in `Store.do` (the `appended` counter) has no test. This covers a `fn` that journals and then errors, and the failed create or remove that retries on the next sync.
- `ClaimAtBoundary` is covered by `TestStore_ClaimAtBoundary` (`verbs_test.go`):
  - Oldest first, agent and program mail, and that Ack marks read: `HandsOutAgentAndProgramMailOldestFirstAndAckMarksItRead`.
  - A released claim leaves the mail unread: `ReleasedClaimLeavesTheMailUnread`.
  - The budget and at-least-one rule: `StopsAtTheBudgetButHandsOutAtLeastOne`.
  - Hold and pause: `UnderAHoldOnlyTheOwnersMail`, `UnderAPauseNothing`.
  - An unreadable pause record and a non-strand sender: `UnreadablePauseRefuses`, `NonStrandSenderRefusedNamingRead`.
- The budget test only pins the first claim as 1 line. The posted lines are 5000, 5000 and 10000 bytes plus a prefix of about 20, so the first two cost about 10040 bytes against the 8192 budget. Whether the joined newlines count toward the budget is not pinned, and no row sits at the budget boundary.
- The `claim` verb dispatch and the ack path through `handOut` are pinned by `ClaimHandsOutTheStrandsMailAndItsFlagFollowsTheMail` (`server_test.go`). That row does not show that a non-strand caller is refused over the socket, but `NonStrandSenderRefusedNamingRead` covers the store side.
- The pane reservation is pinned by `NoticeBeingTypedIsNotClaimedAtABoundary` (`deliver_test.go`). Four related behaviours are not pinned:
  - `releaseNotice` after a busy send, where the notice is available to a hook claim again.
  - `typeNotice` typing nothing when a claim took the notice first.
  - A claim never releasing `paneReserver`.
  - `releaseNotice` answering a wait for the receiver's mail.
- `hook-read` is pinned by the subtest "as a strand hook-read prints its mail as the engine's tool-boundary context, nothing when none is left, and refuses to stderr" (`creelcli/cli_test.go`). The 10-second timeout and exit code 1 are not asserted separately, and the table does not show an unknown-engine row.
- `ToolBoundaryRenderer` is pinned by `TestToolBoundaryRenderer` (`shuttleengines_test.go`). `Claude.ToolBoundaryContext` and `hookContext` are pinned by `TestHookContext`.
- The `PostToolUse` hook appears only in a hub and has no matcher. `TestBuildSettings_CreelMailHook` has rows `InAHub` and `Standalone`. Its `InAHub` row compares against `creelMailHookCommand(...)` computed by the code under test, so it pins the wiring and not the command text. The command text is pinned by the shell test.
- Under `sh` the hook exits 0 without `lym` when the flag is absent or the name is empty, and runs `lym` when the flag stands. `TestCreelMailHook_RunsLymOnlyWhenTheStrandsFlagStands` has rows `FlagStands`, `AnotherStrandsFlag`, `EmptyName` and `UnsetName`.
  - No row uses a name with several segments plus a hyphen beyond `ly:…:driver`.
  - No row has a flag directory containing a space or quote. The quoting is only checked textually in the `InAHub` row.
- `claudeengine.Prepare` wiring (the `c.dirs.Board` → `CreelFlagDir` → Windows POSIX conversion path) has no test. The settings tests call `buildSettings` directly, so a `Prepare` that never passed the directory would go unnoticed.
- `hubgeom` `FlagDir` is pinned by the `TestCreelGeometry` change in `hubgeom_test.go`. The `lymdirs.CreelFlagDirName` constant is covered through it.
- The `buildSettings` signature change is covered by the mechanical argument additions in the existing settings tests. `TestBuildSettings_MatchesGolden` was updated (hunk at `@@ -825`), so the standalone document is unchanged.

**Test economy findings**
1. LOW, `design`: `Prepare`'s hook wiring has no test. The new `creelFlagDirForHook` path in `claudeengine.go` (hub versus standalone, board directory to flag directory) is the only place the engine is "told a hub board directory". No test goes through `Prepare`, so the criterion "a hub Claude strand's settings carry the hook; a standalone one does not" is pinned only at `buildSettings`. A test that runs `Prepare` with and without a board directory would pin it.
2. LOW, `design`: the daemon-start reconcile has no test. `server.go` `Run` calls `store.ReconcileFlags()`, and the acceptance criterion says a daemon start reconciles. `flags_test.go` rows call `ReconcileFlags` directly, so removing the call from `Run` breaks no test.
3. LOW, `design`: the `releaseNotice` busy path, `typeNotice` yielding to a prior claim, and `Store.do` syncing after a `fn` that journaled and then failed have no test. These behaviours are added with no coverage. Only the happy reservation in `NoticeBeingTypedIsNotClaimedAtABoundary` exists.
4. LOW, `design`: there is no return row in `TestStore_MailFlags`. The acceptance criterion names returning the last unread message as a flag-removal path, and the table has post, read, forward, withdraw and program rows only.
5. NIT, `design`: the budget row asserts only the first claim as 1 line and does not sit at the budget boundary. The joined-newline accounting, which `MessageLine` lengths alone ignore, is not pinned.

No new test duplicates one already in its package. Each new row covers a behaviour with no prior test, and `TestHookContext` extends the existing `SessionStart` rendering by row, not by a parallel test. The shell-level `TestCreelMailHook_RunsLymOnlyWhenTheStrandsFlagStands` and the `InAHub` settings row overlap only in having the command in view. The first pins behaviour and the second pins presence, so neither is redundant.

**Tool calls (2)**
1. `git diff` of the test files, filtered to function, subtest and row names. My context held only the non-test diff.
2. `grep` for `ReconcileFlags` in tests, `CreelFlagDir`/`PostToolUse` in claudeengine tests, and the diff of `StopsAtTheBudget` in `verbs_test.go`. My context lacked the test bodies for those.
