```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff changes, find the test row that fails if that behaviour breaks.

**What pins each behaviour:**
1. **Freedesktop hub launcher now has `Name=lym: &lt;shortname&gt;`.** Two rows pin it with exact string equality against a literal: `TestAdd_WritesLaunchersRunningMachineStart`'s Freedesktop row (internal/desktoplauncher/desktoplauncher_test.go:105) and its `%$x\` escaping row (desktoplauncher_test.go:118).
2. **WSL `.cmd` hub launcher now has `rem lym: &lt;shortname&gt;`.** Pinned by the row with a distro (desktoplauncher_test.go:127) and the row without one (desktoplauncher_test.go:136), both exact equality against a literal.
3. **Native Windows `.cmd` hub launcher now has `rem lym: &lt;shortname&gt;`.** Pinned by the FormWindows row (desktoplauncher_test.go:167).
4. **The all-hubs launcher still reads `lym: start every hub`.** The diff keeps this, but it now goes through the renamed `title` parameter. It is pinned for Freedesktop by `wantAll` (desktoplauncher_test.go:109) and for WSL by `wantAll` (desktoplauncher_test.go:129). Native Windows has no `wantAll` row, but it builds the text with the same `cmdFile` call (desktoplauncher.go:166), and the WSL row already pins that call.
5. **The `content` and `cmdFile` parameter rename from `purpose` to `title`, and their doc comments.** This changes no behaviour, so there is nothing to test.
6. **The new `doc.go` line on the displayed names.** It is static prose, so it is exempt.

**Findings:** none. Every changed behaviour has a row asserting the exact output against an independent literal. No test is tautological, and no assertion was loosened: only the expected strings changed, and only on the hub-name text.

**Tool calls:** none. My inherited context already held the diff, the rows of `TestAdd_WritesLaunchersRunningMachineStart` and `desktoplauncher.go`.
