```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F2** (verb comments reworded to name no engine): the new text is true for both engines. It names no symbol that could drift, so there is nothing to challenge.
- **F3** (custom binary in `TestPrepare_CreelHooksInAHub`): the waiter command carrying the binary sits in the Stop setting. `creelWaitHookCommand` at `command.go:198` is added with `hookSetting(event, ..., asyncHookEntry(creelWaitHookCommand(creel), ...))` at `command.go:209`, for `hookEventStop`. The test's filter on `hooks.Stop=` therefore covers it. A hard-coded `codex` in `Prepare` would now fail, because the expected line carries `/opt/codex/bin/codex-custom` and the actual one would not.
- **F4** (indirections removed): the double-quote and no-line-break facts from the deleted `creelTrustSentence` comment survive in the `CreelTrustSentence` doc at `creelhooks.go:12-14`. The `init` panic check still guards the line-break case in `claudeengine`. The `strandFlagName` removal drops a comment that the doc of `creelMailHookCommand` repeats, since it says the name is formed by shell parameter expansion and that an empty strand name exits. The doc on `StrandFlagNameShell` in `shuttleengine` was not read.

Further tool calls, 2 of 6:
1. Grep for `CreelTrustSentence` in `internal/shuttleengine/*.go` and for `.bin` uses in `codexengine`. My context lacked the new home's doc comment and the waiter's use of the binary.
2. Grep in `command.go` for which hook event carries `creelWaitHookCommand`. My context lacked the test filter's coverage of the waiter setting.
