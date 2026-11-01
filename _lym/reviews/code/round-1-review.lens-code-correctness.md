```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. List of branches run, one case each:**

- `headlessClaudeCommandPattern` (`internal/shellguard/rows.go:131`), `claude -p 'write x'`: denied. Correct.
- Same pattern, `claude --print=json hi`: denied, because `=` is outside the end-class. Correct.
- Same pattern, `claude --model sonnet -p hi` and `claude "a b" -p`: denied, since the argument run backtracks and covers quoted words. Correct.
- Same pattern, `/usr/local/bin/claude -p hi`, `claude.exe -p hi`, `cd x && claude -p hi`, `echo x | claude -p hi`, `env claude -p hi`, `sudo claude -p hi`, `claude -p` (JSON-closing quote after the flag): all denied. Correct. I ran an ERE approximation with `grep -E` for the argument, pipe, env, sudo and bare-`-p` cases.
- Same pattern, `claude --print-x hi` and `claude --version && ls -p`: allowed. The `-` end-class and the `&` separator stop the match. Correct.
- Same pattern, `claude -pc hi` and `claude -cp hi`: allowed. These are combined short flags where `p` is followed or preceded by another letter. Not correct; see F1.
- Same pattern, `bash -c`, `X=1 claude -p`, `sudo -u x claude -p`: allowed. The row comment documents this as a guardrail limit. Acceptable.
- `Row` entry, `Switch: SwitchNone` and `Forms`: installed in every run mode. Correct. The `prepare_test.go` counts of 5 (no refuse-raw) and 6 (with refuse-raw) match the one added row.
- `claudeDenies()` in `settings.go`: the Agent rows are removed, and the webster fork guard is untouched and still keyed on `forkSubagents`. Correct.
- `buildSettings` / `buildDenyNotice` / `denyInputs`: the `allowAgentTool` parameter is gone, and the claudeengine and test call sites agree on the new arity. Correct. The repo-wide grep shows no remaining `DenyAgentTool`, `AllowAgentTool` or Agent steer constants in code.
- `ConfigSpec` `RetiredKeys` and `RetiredWayForward` for `deny_agent_tool`:
  - A repo-default layer holding the key is dropped by `dropRetired`.
  - A local layer holding it is refused with the way forward (`internal/configengine/layers.go:279-286`).
  - Both behave as intended.
- `TestBuildSettings_DenyToggleMatrix` table rows: the field order matches the struct for every row. `len(preToolUse) != 5` is guarded by `!tt.fork`, which is correct because the fork guard adds an entry.
- Golden file: 16 unique `interactive=… fork=… denyAsk=… denyPython=…` labels, no duplicates, no leftover `allowAgent` or `denyAgent` text. This matches `mask < 16` and `goldenLabel`. Correct.
- `orchSpec`: dropping `AllowAgentTool` leaves `ForkSubagents: true`, and no hook now denies the Agent tool. Correct.
- `TestSmokeBurlerClusterRogueFork`: it now relies on the post-hoc `auditClusterRound` alone, since no hook refuses the fork's Agent call. Correct for the Goal.

**Findings:**

F1. LOW, class `design`, `internal/shellguard/rows.go:131`.
- The headless-claude pattern misses a print flag bundled with another short flag (`claude -pc hi`, `claude -cp hi`). It needs `-p` or `--print` as a whole word, and `-p` followed by a letter is allowed.
- The focus asks whether combined short flags are refused, and they are not. The row comment lists no such limit, and `rows_integration_test.go` has no row for it.
- Evidence: I ran the pattern with `grep -E` on payload lines for both forms and neither matched.
- Fix: either widen the flag alternative to also match a short-flag cluster containing `p` (a word `-[A-Za-z]*p[A-Za-z]*`), with deny and allow rows, or add the bundled-flag form to the row's "guardrail, not a barrier" list.

F2. LOW, class `consistency`, `internal/configreg/keymove.go:20`.
- The move `shuttle.claude_deny_agent_tool → claude.deny_agent_tool` still targets a key that claude's `ConfigSpec` now retires.
- A reconcile writes the retired key into `claude.yaml`. A hub or worktree layer drops it on load, but a local layer is refused with the way forward.
- This was not exercised end to end.
- Fix: drop that `KeyMove` row, so the shuttle spec's own retired-key handling covers `claude_deny_agent_tool`. Also check `configsync_test.go` against the row's removal.

**Tool calls:**
1. Read `internal/shellguard/rows.go:55-109`: the manifest lacked the definitions of `commandPositionPattern` and `shellArgumentPattern`.
2. `grep` over `settings-golden.json`: the manifest did not show the golden file's labels.
3. `printf | grep -E` with an expanded approximation of the headless pattern: to run concrete command lines through it. The approximation drops the double-quoted-argument alternative, so ignore its non-match on `claude \"a b\" -p`.
