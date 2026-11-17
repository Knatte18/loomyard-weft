---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/08-plugin-session-start-contexts.md (Intent, RunPluginHook paragraph; Tests, TestRunPluginHook)
    summary: >-
      TestRunPluginHook's timeout row asserts a measured wall-clock bound a loaded machine can reach,
      and the card names no cmd.WaitDelay value, so the row neither fits its own bound nor pins the group kill.
    fix: >-
      In card 8, name the WaitDelay as a stated constant in the Intent (for example hookWaitDelay, two seconds)
      beside the group kill, so the bound the card promises has a value.
      Replace the timeout row's text "sleep 5; true under a 100ms timeout returns an error well within a second"
      with: the command "sleep 120; true" (its shell forks the sleep) under a 100ms Timeout returns an error naming the timeout,
      and the call returns before a generous 60 second deadline checked with a select on a done channel, never a measured elapsed bound of one second.
      Add a second assertion that pins the group kill by outcome, not time:
      the command is "(sleep 2; touch $MARKER) & wait", MARKER is a path under t.TempDir() passed through Env,
      and after the timeout error the test waits 3 seconds and asserts with os.Stat that the marker never appeared,
      since a surviving forked child would have created it.
      Keep both rows in pluginhook_integration_test.go, which already names tier1 for every function.
      No target is added, dropped or moved, so no validate-plan overlay is needed.
  - id: F2
    severity: LOW
    class: decision
    location: _lym/plan/07-claude-user-setup-source.md (Intent, InstructionFiles bullet; Tests, TestUserSetup_InstructionFiles) and _lym/plan/01-claude-imports-leaf.md (Tests, TestExpand)
    summary: >-
      The unreadable-file rows of cards 7 and 1 name no input that makes a file unreadable,
      and the obvious input, a mode-0 file, reads fine under root, so the row fails on a root gate run.
    fix: >-
      In card 7's TestUserSetup_InstructionFiles, name the input for both unreadable rows (the global file and a CLAUDE.md above the root)
      as a directory standing at the CLAUDE.md path: os.ReadFile of it fails with a non-ErrNotExist error for every user, root included,
      so the row needs no skip and still tells an unreadable file from an absent one.
      In card 1's TestExpand, name the same input for the unreadable path row.
      As an alternative, state a mode-0 file with the row skipping itself when the test runs as root, as card 10 states for its unreadable directory row.
---

### [BLOCKING] TestRunPluginHook's timeout row asserts a load-reachable time bound and the card names no WaitDelay

**Location:** `_lym/plan/08-plugin-session-start-contexts.md`, the `RunPluginHook` paragraph of the Intent and the `TestRunPluginHook` bullet of Tests.

**Issue:** Card 8 plans a tier1 row where `sleep 5; true` under a 100ms timeout "returns an error well within a second".
`PATTERN-test-speed`'s "Time is injectable" section bars a test from comparing a measured real time to a bound a loaded machine can reach.
It allows a proof that a call does not wait only when the awaited thing outlasts a generous deadline, a minute, by a release the test controls.
A one-second bound over a spawned `sh` and `sleep` under a parallel tier1 gate is reachable by load, and the five-second sleep outlasts it by a factor of five, not a generous margin.
The rule is a listed constraint, so the plan's own planned test violates it.

The same row does not pin what the card says it pins.
The card adds `setHookProcessGroup` and a `cmd.Cancel` group kill "so that the bound holds when the command's shell forks a child that keeps the output pipes open", and it also sets `cmd.WaitDelay`.
`WaitDelay` alone returns the call once the delay passes after the context ends, with the forked `sleep` still alive and holding nothing the call waits on.
So the row passes with or without the group kill whenever the `WaitDelay` is under the bound, and the card names no `WaitDelay` value, so no reader can tell whether the one-second bound is consistent with it.
The leaked `sleep` is the failure the group kill prevents, and no planned assertion observes it.
The decision's bound for the hook run (D8: a hook that times out is left out) is carried, but the card's mechanism is untested and its time bound is unstated.

The remaining parts of this round's first focus question are settled and are not defects:
`TestRunPluginHook` is the only planned test that starts a real process through `RunPluginHook`, it is stated tier1, and its file `pluginhook_integration_test.go` names tier1 for every function, as `PATTERN-test-speed` requires of a file.
`TestUserSetup_PluginSessionStartContexts` uses a fake `RunHook` and is stated tier0, which matches the rule that a tier0 test spawns nothing.
Card 9's `TestNew_CodexLaunchCarriesTheClaudeUserSetup` is stated tier0 and wires the real `RunPluginHook`, but its config directory holds no plugin, so no hook runs and no process starts.

### [LOW] The unreadable-file rows of cards 7 and 1 name no input

**Location:** `_lym/plan/07-claude-user-setup-source.md`, the `InstructionFiles` bullet and the `TestUserSetup_InstructionFiles` bullet; `_lym/plan/01-claude-imports-leaf.md`, the `TestExpand` bullet.

**Issue:** The fourth focus question asks whether the refusal for an unreadable global file and the warning for an unreadable above-root CLAUDE.md have a stated bound, a named input and a named test.
The bounds are stated: the global file's read error is the one error `UserSetup` returns, with a way forward, and an unreadable CLAUDE.md above the root is left out with a warning "since the walk reaches machine directories the user may not own".
The tests are named: both rows sit in `TestUserSetup_InstructionFiles`, and the code they reach is the `os.ReadFile` error branch of `instructionFiles`.
The input is not named.
"A global file that cannot be read" and "an unreadable CLAUDE.md above the root" do not say how the file becomes unreadable.
The usual construction, `os.Chmod(path, 0)`, leaves a file readable to root, and card 10 already treats that case for its unreadable directory row by skipping when the test runs as root.
Card 7 states no such skip, so a root gate run fails both rows.
`PATTERN-test-speed` allows a skip that names a missing prerequisite, but it bars a skip by design, and a hermetic input exists.
A directory standing at the CLAUDE.md path makes `os.ReadFile` fail for every user, and the error is not `ErrNotExist`, so the row also pins that an unreadable file differs from an absent one.
Card 1's `TestExpand` row "an unreadable `path` returns an error" has the same gap, and a missing file would also satisfy it, though card 1's own import rows already say a missing import stays as written.

The process-ownership half of the same focus question, in card 8, is not a defect and is listed under Rejected.

## Rejected

- Card 5's argument-size boundary.
  `MAX_ARG_STRLEN` is 131072 and counts the terminating NUL, so `execve` accepts an argument of at most 131071 bytes.
  Run in this session, `/bin/true` took a 131071-byte argument and refused a 131072-byte one with "Argument list too long".
  The card refuses a setting of at least `maxArgumentBytes` bytes and lets `maxArgumentBytes`-1 go ahead, which is the exact `execve` boundary.
  The launch line reaches the pane as a sourced script (`stageLaunchScript` in `internal/reedengine/panebin.go`), not as a `send-keys` payload or a `sh -c` argument, so only the setting itself is one `execve` argument.
  The `AtTheArgumentCap` row (`maxArgumentBytes`-1, goes ahead) and the `OverTheArgumentCap` row (exactly `maxArgumentBytes`, refused) sit on the correct sides.
  The record's wording "passes 128 KiB" is met, since the card refuses everything `execve` would refuse.
- Card 10's list of whole-line Prepare tests.
  In `internal/shuttleengine/codexengine`, the tests that compare a whole `Launch.Cmd` are `TestBuildLaunchCmd`, `TestPrepare_Launch`, `TestPrepare_SandboxMode`, `TestPrepare_ExpandsHomeInWritableRoots` and `TestPrepare_InstallsGuardsFromItsInputs`, plus card 6's `TestPrepare_SubagentTerms`, and card 10 edits every one of them.
  `TestPrepare_SkillsLoadFirst`, `TestPrepare_CreelHooksInAHub`, and the cards 5 and 9 tests that read the developer instructions use `strings.Contains` on the line, not a whole-line comparison.
  `TestOpenLaunch` compares a `codexCommand` line built from a `launchLine` that leaves the new fields zero, so its want is unchanged.
  No test outside `codexengine` compares a Codex launch line.
  The fixture worktrees hold no `AGENTS.md`, `AGENTS.override.md` or `CLAUDE.md`, so `defaultProjectDocMaxBytes` is the right want for the edited tests.
- Card 8's hook processes left by a killed `Prepare`.
  The bound is stated (the timeout and group kill hold only while the lym process lives, then the hook runs to its own end, review-held), the card says why no `runproc` record exists, and I confirmed `shuttleengine.Spec` and `Config` carry no owner and no `internal/shuttleengine` file imports `runproc`.
  The existing `runSandboxProbe` in `codexengine/sandbox.go` starts and waits on a process the same way with no record, so the plan follows the package's practice.
  This is a documented gap, not a prescribed refusal, so the reachability bullet does not apply.
- Card 8's sequential hook runs have no total bound, only a per-hook one.
  D8 sets the per-hook bound and licenses a plugin's own `timeout`, so the plan carries the decision.
- Developer-instructions order in the overview.
  The record's parenthetical lists the subagent passage first, but the record's rule is "in that file's order", and `appendPromptText` with `claudeengine.go:263` puts the deny notice and creel sentence first, then the subagent passage, then the standing passages.
  The plan follows the file, which is the record's rule.
