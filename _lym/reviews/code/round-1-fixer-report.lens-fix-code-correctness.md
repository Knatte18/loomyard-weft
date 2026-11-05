```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. Task: fix-code-correctness lens on the F2, F3 and F4 changes.

- **F3 test, defect in place:** `hookSettings` puts `creelWaitHookCommand(creel)` into the `hooks.Stop=` setting (`command.go:209`), and that command spells `LYM_CODEX_BINARY=&lt;posix-quoted creel.bin&gt;` (`command.go:198`). `TestPrepare_CreelHooksInAHub` checks every `hooks.Stop=` setting against `launch.Cmd`. With the fixture's `binary: /opt/codex/bin/codex-custom` and a `Prepare` that hard-coded `codex` (`codexengine.go:132`), the launch line would carry `LYM_CODEX_BINARY=codex`. The expected setting carries `LYM_CODEX_BINARY=/opt/codex/bin/codex-custom`. The `strings.Contains` check then fails, so the test pins the binary.
- **F3 fixture:** `writeCodexConfig` appends `extra` after `home:` and `operator_home:`, one key per line. `binary: ...\n` is therefore valid YAML under the `yaml:"binary"` tag (`config.go:30`). The custom binary also reaches the launch line's `sh.Invoke(line.bin)`, and the test never builds that line itself, so nothing else in the test depends on the old default.
- **F4:** The change replaces `strandFlagName()` with `shuttleengine.StrandFlagNameShell()` at both call sites (`settings.go:118` and `:147`, before the edit), which gives the same string. It replaces `creelTrustSentence` with `shuttleengine.CreelTrustSentence` in `appendPromptText`, the `init` guard, `settings_test.go` and `prepare_test.go`, and it was a plain const alias, so values are identical. The post-edit grep finds no remaining reference to either deleted name, `go vet` compiled all three packages, and `lym gate test` passed tier 0 (905 passed, 0 failed).
- **F2:** Comment-only change, no behavior to run.

Tool calls: 2.
1. `grep` over `codexengine/*.go` non-test files, to see where `creel.bin` and `codexBinary` are used; my context lacked `Prepare`'s non-test source.
2. `Read` of `command.go` lines 190–250, to confirm the waiter command sits inside the `hooks.Stop=` setting the test checks; my context lacked `hookSettings`.
