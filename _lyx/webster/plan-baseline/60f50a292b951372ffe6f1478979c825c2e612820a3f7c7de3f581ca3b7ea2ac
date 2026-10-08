# Card 2 — python-deny

**Edit:**
- `internal/shuttleengine/config.go`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/template.go`
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/config_test.go`
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/norecoverydeny_test.go`
- `pattern/PATTERN-no-denied-recovery.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`

**Intent:**
Add the `python` deny (D11) as a new row of card 1's deny table, switched by a new `shuttle.yaml` key `claude_deny_python` (`Config.ClaudeDenyPython`, template default `true`, with a template comment naming what it denies and that a Python target repo turns it off).
An existing `shuttle.yaml` without the key reads the template default, through `configengine`'s missing-key fill.

The row's matcher is `Bash`, installed for every lyx-spawned session whenever `ClaudeDenyPython` is true: interactive, autonomous and fork-mode alike, and independent of `allowAgentTool`.
Its hook command greps the raw PreToolUse payload JSON with `grep -Eq`, like the webster fork guard, and echoes the deny JSON on a match, ending in `; true` so a non-match exits 0 and allows the call.
The pattern matches `python`, `python3` or `python3.<minor>`, optionally path-prefixed (`/usr/bin/python3`), as a whole word in command position: at the start of the `command` value (right after `"command":"`), or after a JSON-escaped newline (`\n`), `;`, `&`, `|`, `(`, `$(`, a backtick, or one of the words `env`, `exec`, `xargs`, `sudo`, `time`, each with optional whitespace before the program.
It lets through, as D11 accepts: a `python` word as an argument (`grep python file`, `ls python-tools/`), Python behind `bash -c`, `sh -c` or `eval`, after a prefix not in the list (`env FOO=1`, `nohup`, `command`, `timeout`, a shell keyword), through a launcher (`py`, `uv run`, `uvx`, `pipx run`), and any other runtime.
It falsely denies `python` after a listed separator inside a quoted argument, a heredoc body or another payload field such as `description`; this is the guardrail-not-barrier bound, and the doc comment on the row says so.
Its steer: lyx agents never run Python; edit files with Edit or Write, search and read with grep, awk or cat.
Its notice announces it in the same wording, and is appended after today's sentences, so the notice differs from today's only by that sentence; both pass the `init` forbidden-character check.

What the deny lets through is bounded above; it adds no way to skip an existing deny, and `claude_deny_python: false` removes only this row.

Tests:
- `settings_test.go`, as rows and dimensions of existing tests: `TestBuildSettings_DenyToggleMatrix` gains a `claude_deny_python` column and rows showing the python hook installed for interactive, autonomous and fork-mode sessions and absent under `claude_deny_python: false`; `TestBuildDenyNotice_MatchesInstalledDenies` gains the python switch as a further loop dimension, so the notice carries the python sentence exactly when the hook is installed; every existing row's expected document and notice gains the python entry or sentence where its config enables it, and stays otherwise byte-identical.
- `prepare_test.go`: `TestPrepare_AppendSystemPromptMatchesDenyNotice` gains the python switch as a further input dimension.
- `config_test.go`: `TestLoadConfig_TemplateDefaultsResolve` asserts the shipped template reads `ClaudeDenyPython` true.
- `hookcmd_integration_test.go` (`integration` tier, run through the package's existing `tmuxkit.Main`): one table-driven function runs the python hook command under `sh` against PreToolUse payloads built with `encoding/json`, and asserts the deny JSON for `python3 -c 'print(1)'`, `cd x && python script.py`, a second line starting `python3`, `/usr/bin/python3 - <<EOF`, `xargs python3`, and empty output with exit 0 for `grep python file`, `ls python-tools/`, `bash -c 'python3 x'` and `uv run x`.
- `cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine/claudeengine` row rises by 1, for the one new top-level function in `hookcmd_integration_test.go`.

Docs: `cmd/lyx/norecoverydeny_test.go`'s `deniedRecoveryForms` gains `{"python"}` and `{"python3"}` with a self-test row offering `python3` as a way forward that fails; `pattern/PATTERN-no-denied-recovery.md` adds `python` and `python3` to the closed list and names `claudeengine`'s deny table in `settings.go` as the owner of lyx's own denies, replacing the `agent-permission-setup` pointer, and the same pointer in `norecoverydeny_test.go`'s comment.
`template.go`'s `ConfigTemplate` comment lists `claude_deny_python` among the plain-literal keys, and `spec.go`'s `Spec.Interactive` comment says the python deny, like the Agent deny, is installed in both modes and names `claude_deny_python` beside the other deny keys.
`claudeengine/doc.go` describes the deny table and the python row; `docs/overview.md`'s shuttle entry names the python deny and its `claude_deny_python` switch beside the Agent and AskUserQuestion denies.

**ImpactSummary:** Every lyx-spawned Claude session gains a Bash PreToolUse python deny and a notice sentence by default; shuttle.yaml gains claude_deny_python; the denied-recovery scan list grows by python and python3.
