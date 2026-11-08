```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. Status job and toggle click never run a test binary: `bindings_test.go:263` (row `TestBinaryExecutable`) and `slacktoggle_integration_test.go:18` (`TestSlackLines_RealTmuxBootByATestBinaryPinsNeither`).
2. Slack lines pinned and unset apart from the other four lines: `bindings_test.go:245` (`TestPinBindingsLocked_PinsTheSlackLinesOnlyWhenEverythingResolves`).
3. Click-dispatch Slack branch bound only on its own flag: `bindings_test.go:110`.
4. `TestMain` exits with status 2 on any positional argument: **none**.
   - `testmain_test.go:19` has no test that runs the guard.
   - The integration test at `slacktoggle_integration_test.go:18` proves only that a test binary's server holds no Slack line.
5. `composeSlackToggleCommand` / `composeSlackStateCommand` line content (quoting, `##` doubling, `slack bar`): **none seen**.
   - `bindings_test.go:281` calls the composers to compute the expected lines, so a change to either composer would not fail it.
   - Only `TestComposeSwitchCommand` (`panebin_test.go:358`) is a content test, and it covers the switch command alone.
6. Toggle formats in the status bar, and `status-interval` pinned to 5: `bar_test.go:44` covers the toggles after the follow button. The `status-interval` pin is probably in `windowsize_test.go:65`, which I did not open, so unconfirmed.
7. `ShowToggleOutcome` (message escaping of `#` and `%`, refresh): `slacktoggle_test.go:52`.
8. `lym slack bar` state line: `toggle_test.go:19` (`TestBar`), covering lit, unlit and unreadable states.
9. `lym slack toggle slack|away`, the switch-off ending every live hub's relay, partial failure and the unknown-toggle refusal: `toggle_test.go:81`, `:152` and `:137`.
10. `Connected`, switch file and presence per machine: `files_test.go:22`, `:50` and `:93`.
11. `SlackMachine` paths: `hubgeom_test.go:570` and `:496`.
12. `EnsureRelay` returning `OutcomeSwitchedOff`: `relay_test.go:1265`.
13. Auto-away before forwarding: `inbound_test.go:1249`, with the failed-write case at `:1289`.
14. Presence notice sent by each relay on a change by any route: `relay_test.go:942`.
    - Notice to the hub's orch alone when the beams read fails: **none seen**.
    - Notice on the `diskCheck` fallback path with no watch (`noticed` differing from the presence): **none seen**.
    - `2065f578`'s message names this fix, so a test may exist that my grep did not match.
15. Hook, prompt side (slash command, `[creel]`, `[lym]`, task notification, inject match for sending, injected and unconfirmed events, no match, unreadable journal, outside an orch, terminal presence): `hook_test.go:41`, rows at lines 64–90.
    - The `injectMatchWindow` boundary (2 minutes): no row visible.
    - Whether the `Since` filter is passed to the journal: no row visible.
16. Hook, Stop side (forwards the trimmed reply, empty reply, terminal presence, post failure, subagent): `hook_test.go:93–110`.
17. `fitReply` including a rune-boundary cut: `hook_test.go:158`.
18. `TypesInject`: `journal_test.go:575`.
19. `HookInput` `Prompt` and `Reply`: `capabilities_test.go:70–74`.
20. Turn hook wiring in settings (prompt and Stop commands, exit status kept, merged creel Stop runs it only on the recording path): `settings_test.go:357` plus `hookcmd_integration_test.go:314`, `:367` and `:415`.
21. Orch spec sets `TurnHookCommand: slackengine.HookCommand`: **none seen** in `orchcli/start_test.go`.
    - The diff shows `start_test.go` gaining 3 lines. They may assert it, but the grep shows no test name for it.
22. `<machine>::<sender>` sender names: `agentname_test.go:272`, and the `orchSender` / `programSender` helpers in `outbound_test.go:108–116`. The relay's own line (`relaySender`) is covered through those helpers.
23. `WritePresence` leaving an unchanged value unwritten: `files_test.go:93` (assumed) and `relay_test.go:828`.
24. Pinning of `slackToggleCondition` and the `slack` / `away` range names: `bindings_test.go:114`.
25. `role-slack.md` and `orch-role.md` text: static prose, exempt.
26. `slackLineExecutable` branch where `os.Executable()` errors, or the stub differs from the test binary's own path: `bindings_test.go:263` covers the equal-path refusal. The error branch has no row.

**Findings**

- **MEDIUM, design.** The `TestMain` guard (the exit-2 on a positional argument that the parent's amendment item (2) requires) has no test. Location: `internal/reedengine/testmain_test.go:19`. Evidence: no test function or subtest in `reedengine` runs the test binary with an argument. Fix: add a test that runs `os.Executable()` with a positional argument and asserts exit status 2 and that no suite output appears, or state that it is untestable.
- **MEDIUM, design.** Nothing pins what the composed Slack toggle line and state line contain. Location: `internal/reedengine/panebin.go:342` and `:363`, tests at `bindings_test.go:281`. Evidence: the pinned-line test derives its expectations by calling the composers. Fix: add a table test in `panebin_test.go` that asserts the exact toggle line (`slack toggle #{q:mouse_status_range}`, `--socket`, `--tmux`, `--client`, `##` doubling of a path holding `#`) and the exact state line (`<exe> slack bar`) on at least one shell dialect, plus the rows "executable error returns false" and "test binary returns false".
- **LOW, design.** The orch spec's `TurnHookCommand` is not visibly asserted. Location: `internal/orchcli/start.go:210`. Evidence: no test name in `start_test.go` mentions it. Fix: assert `spec.TurnHookCommand == slackengine.HookCommand` in the existing orch spec test, unless `start_test.go`'s 3 added lines already do. Check before adding.
- **LOW, design.** Presence notice paths with no visible row: the beams-read-failure notice to the hub's orch alone (`relay.go:641–654`, from `2065f578`) and the `diskCheck` fallback path (`relay.go:604`). Fix: add rows to the `TestRunRelay` presence-change subtest. For the first, make `Beams` fail while the presence changes and assert one notice, to the hub's orch. For the second, make `Watch` return an error and assert a notice still goes out after a presence change.
- **NIT, design.** The hook inject window is unpinned. Location: `internal/slackcli/hook.go:28` and `:140`. Fix: add a row to `TestHook` with an inject journaled more than `injectMatchWindow` before the prompt, expecting presence terminal.

**Tool calls (2)**
1. A grep for test function names across the diff's test files, to find which behaviours have a named test. My context held no test file.
2. A grep for subtest and row names in the hook, toggle, bindings, relay, outbound, capabilities and `cli_test` files, plus `TestMain` mentions, to map rows to behaviours.
