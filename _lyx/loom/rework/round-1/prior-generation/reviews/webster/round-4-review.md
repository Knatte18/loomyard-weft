---
verdict: APPROVED
---

No findings.
The range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main` in `_lyx/shed/shuttle-send/status.json`; no `_lyx/loom/rework/` round exists).
The round's focus is the round 3 fix, commit `00b9be707`, and the verdicts below settle its two questions.

## Focus 1: do the loomcli stub providers satisfy both StartupReady and IdleSession before every verified send

Not a defect.

- `awaitIdleSession` (`internal/shuttleengine/send.go`) needs `Startup(capture) == StartupReady` and `IdleSession(capture)` on the pane, and `IdleSession` reads the box under the last `❯` line.
- `claudeengine.IdleInputBoxFixture` is a top rule, an empty `❯` line, a bottom rule and the `? for shortcuts` footer.
  The `ready_idle_input_box_fixture` row of `TestStartup_Classification` pins it as `StartupReady`.
  The `exported idle input box fixture` row of `TestIdleSession` pins it as idle.
- Both stub scripts (`integrationWriteStubDriverScript`, `writeStubDriverScript`) draw the fixture at launch and again after each line they read, so a new box sits below the previous echo before the next send.
  The caret search takes the last `❯`, so the older boxes above it do not matter.
  - First send: the launch box.
  - Skill-load message: the box drawn after the `/color` line, or after the load message's own preceding read when no `/color` is typed.
  - Pointer send after the skill-load turn: the box drawn after the load message is read, before the `Stop` event is appended and the next `read` blocks.
- The typed text echoes below the box in cooked mode, so `InputBoxText` reads an empty box.
  The settle check agrees on two empty reads, and `confirmSubmitted` finds the box not holding the sent text after its first Enter.
- The shell quoting holds: the fixture, which contains newlines and `❯`, sits inside single quotes, and the `Sprintf` argument order (fixture, then `settleDelay`) matches the two `%s` slots.
- I ran `go test -tags tmux -count=1 -run 'TestIntegrationDriverBootstrap|TestSmokeDriverStrand' ./internal/loomcli/`, and it passes.
  I also ran `go test ./internal/shuttleengine/...` and it passes.

## Focus 1, second half: any other stub or fake provider that prints only `ReadyFooterFixture`

None.

- `grep` finds no other user of `ReadyFooterFixture` outside `claudeengine`, and its remaining uses are the `IdleInputBoxFixture` definition and one `Startup` classification row.
- The only engines in the tree that implement `SessionCycler`, and so hit the idle wait, are `claudeengine.Claude` and the `shuttleengine` test fakes.
  Every shell-script engine in `internal/loomcli` (`shellLaunchEngine`, `gateRepromptReadEngine` and the parent-review engine) implements no `SessionCycler`, so `awaitIdleSession` falls back to `requireReadyAgentPane` for them and the change does not reach them.
- The live-substrate smokes outside `loomcli` drive a real `claude` binary.

## Focus 2: test economy of the new fixture

Not a defect.
`IdleInputBoxFixture` is pinned by one row in each of the existing tables, `TestIdleSession` and `TestStartup_Classification`.
The diff adds no separate test, so `PATTERN-test-economy` holds.
