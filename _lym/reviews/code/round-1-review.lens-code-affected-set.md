```yaml
lens: code-affected-set
outcome: reported
```

Task: affected-set review of the per-orch Slack button diff. Per change, the set I found and whether it is covered:

1. **`SwitchPath`, `WriteSwitchedOff`, `ReadSwitchedOff`, `OutcomeSwitchedOff` and `SwitchFileName` removed.** A repo-wide grep finds no leftover reference outside deleted lines, `docs/research` and the driftengine matches, which are unrelated. Tests are updated in `hubgeom_test.go`, `cli_test.go` and `relay_test.go`. Covered.

2. **`[slack presence]` notice removed.**
   - No code references remain.
   - `role-slack.md` drops the line.
   - The `slackcli/doc.go`, `slackengine/doc.go`, `presence.go` and `cli.go` comments are updated.
   - Covered.

3. **`ForwardingPath` and `forwarding.json` added.** The relay's wake watch, `ensure`, `status`, `bar`, the hook, `start.go` and the outbound read all use it. `docs/overview.md` lists it. Covered.

4. **`ChannelName` signature and channel map file renamed to `orch-channels.json`.**
   - All callers are in `keeper.go`, and the tests are updated.
   - Covered.
   - A caveat on the old-channel rule: `missingTokenChannel` in `relay.go:333` reads `ChannelMapPath`, so with the new file name its fallback to the older setup file's channel id is the only path left to old channels. That matches the answer key, but see F-1 below.

5. **`Hub` struct gets `Project`, and `newChannelKeeper` gets a `hub` parameter.** Both `runRelay` in `wiring.go` and the test fakes supply them. Covered.

6. **`reedengine` state line and `#{E:` expansion.** `bar.go`, `panebin.go` and their tests are updated. `internal/reedengine/doc.go:236` says "a Slack toggle runs `lym slack toggle slack|away` from the session's run directory", which is still accurate. Covered.

7. **`status` field `connection_off` replaced by `forwarding`.** I found no other reader in non-test code. Covered.

8. **Behaviour of `start`, `away` and `EnsureRelay`.**
   - `fabriccli/doc.go:185`, `orchcli/doc.go:32` and `beamcli/doc.go:474` describe the start paths.
   - None of them mentions the switch, so they are still accurate.
   - The hub prime's bring-up (`orchcli/start.go:595`, `fabriccli/anchordeps.go:283`) now returns `OutcomeForwardingOff` when no button is on. Callers that treat unknown outcomes as errors were not checked; I did not read them within my tool budget.

9. **PATTERN entries.**
   - `pattern/PATTERN-daemon-wakeups.md:93` still describes "the inbox and presence watch in `internal/slackengine/relay.go`". The diff changed that watch to the inbox and the forwarding file, and the presence file no longer wakes anything.
   - `PATTERN.md` is not changed by the diff. The directive declares no PATTERN changes, so there is no "declared changes landed" finding.
   - The daemon-wakeups entry is a stale description, raised as F-1.

10. **Commit messages and departures.** No commit carries a `Pattern-Departure:` trailer, and none is needed.

11. **Out-of-limits files (the focus's scope question).**
    - `boardengine/render.go` adds `RootProjectHandle`. The Limits list names only slackengine, slackcli, reed's Slack toggles, the role file and their tests, so a change to board-module code lies outside it. The decision-record goal needs the root project handle `p0`, and `slackcli` could not otherwise reach `rootProject`. The edit is minimal and has no test of its own.
    - `hubgeom/slackgeom.go` is the geometry wiring for the new `ForwardingPath`. It is a necessary consequence of the change, not a departure.
    - `docs/overview.md` is the doc update the rules require.

**Findings**

- **F-1: LOW, consistency.** `pattern/PATTERN-daemon-wakeups.md:93` still says the relay's wake watch covers "the inbox and presence watch". The diff changes that watch to the inbox plus the forwarding file (`relay.go` `watch()`, `run` doc), and the presence file no longer wakes the relay. Fix: change that bullet to "the inbox and forwarding-file watch".

- **F-2: LOW, scope.** `internal/boardengine/render.go:185` adds an exported `RootProjectHandle` with no test, and board-module code is outside the Limits list. This is a candidate for the handler's scope judgement, not a clear defect: the answer key requires `p0` in the name, and `orchProject` in `wiring.go` is the only caller. It has no direct test; the `hubgeom`/`slackcli` tests use a fake `Project`, so the real `orchProject` path, including a board with no matching project entry, is not covered. The handler or the code-tests lens should decide on that gap.

**Tool calls**
1. A repo-wide grep for stale terms (switch, presence notice, `channels.json`, daemon-wakeups wording, `primeOrchName`). My context lacked a repo-wide search for leftovers outside the diff.
