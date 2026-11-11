```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it or "none". Four behaviours have no test: findings F1 to F4 follow the list.

**Per behaviour**
- The forwarding file's read, write, sorting and repeat-unchanged behaviour, the missing-file case and the undecodable-file error: `TestForwarding` in `files_test.go`. Pinned.
- Restart survival: the state is a file, and `TestForwarding` reads back what it wrote. Pinned.
- `dropForwarding` on retire: the Retire subtest in `keeper_test.go` checks `buttonsOn` before and after, and after a refused archive. Pinned.
- `Renamed` and the retire without a second rename: "AnEntryMarkedRenamedIsArchivedWithNoSecondRename" and `requireArchivedUnder`. Pinned.
- `ChannelName` order, case, characters, empty parts and the length cut: `TestChannelName`. Pinned.
- Project naming from the hub (root `P0 root`, a beam's project, a beam with no board entry, a failed project read): the keeper subtest "ANewChannelIsNamedForTheProjectTheHubNamesForTheOrch". Pinned.
- `ensure` makes a channel only while the button is on, and returns an open entry whatever the button: the keeper subtest "OnlyAnOrchWhoseButtonIsOn…". Pinned.
- Outbound drops mail of an off orch, the redirected gone beam, the stale line with no open channel, and creates no channel: the outbound subtest "MailRoutedToAnOrchWhoseButtonIsOff…". Pinned.
- Inbound answers in the channel and does not forward while the button is off, with `away` still working and the cursor advancing: the inbound subtest "OperatorMessageInTheChannelOfAnOrchWhoseButtonIsOff…". Pinned.
- `ForwardingOn` read failure in `inbound.handle`, which goes to `retryOrGiveUp`: none. See F4.
- Relay start ensures the channel of each listed, pressed orch and skips an unlisted beam: the `TestRunRelay` subtest "RelayStartMakesTheChannelOfEachListedOrch…". Pinned.
- A press of a button wakes the relay, and presence and own-record writes do not: "WritesToTheRelaysOwnRecordsAndToThePresenceWake…". Pinned.
- The `wakeLoop.diskCheck` fallback comparing the forwarding set when the watch cannot open: none. The grep found no test for the watch-failure path in `relay_test.go`. See F1.
- The relay starts nothing with no button on (`OutcomeForwardingOff`), for operator start and while away, and starts once one is on: `TestEnsureRelay` "HubWithNoButtonOn…". Pinned.
- The relay ends when the last button goes off, and stays while another is on: `TestToggle` rows "…LastButton…" and "…WhileAnotherOrchsButtonIsOn". Pinned.
- Toggle refusals (a pair's orch, a failed orch lookup, no login or setup, a relay that won't start keeping the button on, a relay that won't end): `TestToggleSlackRefusals`. Pinned.
- `bar` for each state, no `--dir`, an unknown directory, a failed wire, a failed orch lookup, and an unreadable presence or machine: `TestBar`. Pinned.
- `away` with and without a relay to await: the `TestSlackCLI` subtest "outcomes". Pinned.
- `status` field `forwarding`: `TestSlackCLI` status rows. Pinned.
- Turn-end hook posts only for an orch with its button on, and `button-off` leaves presence alone: `TestHook` "StopWhileTheOrchsButtonIsOff…". Pinned for the prime's orch.
- Hook with a beam orch whose button is on, so the `agentname.TaskSlug(strand)` derivation of the orch name: none. See F2.
- Hook `ForwardingOn` read failure: none. It is a minor hook error path that logs and returns.
- Status-bar slack toggle shown only on `slack-on` or `slack-off`: `TestStatusFormatActions_SlackAndAwayTogglesFollowTheFollowButton`. Pinned.
- State command `slack bar --dir #{@lym_button_dir}` and the `#` doubling: `TestComposeSwitchCommand`. Pinned.
- `ForwardingPath` in the geometry: `TestSlackGeometry`. Pinned.
- Dobby prime tick starts a relay only with a button on: `TestHubDobbyDeps_PrimeTickEnsuresRelay`. Pinned.
- Existing channels keep their names and are left alone because the map file is renamed to `orch-channels.json`: none. `TestSlackGeometry` builds its want from the `ChannelMapFileName` constant, so it cannot catch a revert of the literal. See F3.
- `slackcli.worktreeOrch` (prime, beam or other worktree to an orch name or empty) and `orchProject` (board read, project handle and slug, fallback with an empty handle) in `wiring.go`: none. The harness replaces `worktreeOrch` with a fake, and the grep found no tests for either function or for `boardengine.RootProjectHandle`. See F5.
- No `[slack presence]` mail is sent: the old notice tests were deleted, and no test asserts absence. I raise no finding for a removed feature.

**Findings**

- F1. MEDIUM, design. `internal/slackengine/relay.go:597-613`. `diskCheck` now also compares `ReadForwarding` against `w.forwarding`, and `w.forwardingKnown` guards the first comparison. This is the only wake path for a button press when the watch cannot open, and no `relay_test.go` row exercises it. Evidence: the grep found no watch-failure test, and the diff touches none.
- F2. LOW, design. `internal/slackcli/hook.go:177-187`. The orch name is derived with `agentname.TaskSlug(strand)`. `TestHook` has no row for a beam orch strand with its own button on posting. The only beam row uses `hub:b:orch` as another orch's button.
- F3. LOW, consistency. `internal/slackengine/geometry.go:25`. The old and new map file names are a decision of the task, "existing channels are left alone". No test pins the literal `orch-channels.json` or that a leftover `channels.json` is never read. `hubgeom_test.go` and the harnesses use the constant.
- F4. LOW, design. `internal/slackengine/inbound.go:614-617`. The new error branch (`ForwardingOn` fails, then `retryOrGiveUp` with "the orch's slack button could not be read") has no test row. It is a new failure path in the message-hold logic.
- F5. MEDIUM, design. `internal/slackcli/wiring.go:178-213`. `worktreeOrch` decides which worktrees get a button (prime, beam, not pair) and `orchProject` supplies the channel-name project. Neither is tested, and `cli_test.go:286` fakes the first. The "a pair's orch gets no button" decision and the channel-name project (the root project's `p0-root` handle, a beam's project, the empty-handle fallback) are therefore pinned only through fakes of their own seams.

**Tool calls**
1. A grep of `relay_test.go` for `diskCheck`, watch-failure and fallback terms, and of all `_test.go` files for `worktreeOrch`, `orchProject` and `RootProjectHandle`. My context lacked these because the `relay_test.go` diff did not show unchanged tests and the `wiring.go` tests were not read.
