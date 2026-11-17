```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each thing the diff changes, I looked for everything outside the touched lines that calls, reads or describes it, and checked whether it still holds.

**Per change, the set found:**
- **Removed `orchengine` remote-control record** (`RemoteControlRecord`, `WriteRemoteControl`, `ReadRemoteControl`, `RemoteControlOn`, `Paths.RemoteControlPath`/`RemoteControlLockPath`): callers are `orchroute/paths.go`, orchcli, the `cli_test`/`state_test` fixtures and `orchengine/doc.go`. All are updated, and a repo-wide grep finds no leftover reference. Covered.
- **Removed `rc-bar` verb, `RemoteControlBarVerb`, the `rc-on`/`rc-off` words, `@lym_rc_state_command` and `composeRemoteControlStateCommand`**: references were in `cli.go` PersistentPreRunE, `bindings.go`, `panebin.go`, `bar.go`, both doc.go files and the tests (`panebin`, `bindings`, `windowsize`, `attach`, `bar`, `bindings_integration`). All updated, and the repo-wide grep finds no hit. Covered.
- **New session option `@lym_orch_dir` and the Slack state line's new `--dir`**: affected are `buttonrow.go`, `panebin.go`, `reedengine/doc.go`, `bar.go`, `panebin_test`, `buttonrow_test`, `windowsize_test` and the `slackcli` `bar` verb (`toggle.go`). The `slackcli` `--dir` flag help is not updated (finding 1).
- **New `turnOnRemoteControl` signature and click behaviour** (reads `@lym_strand_name` and `@lym_rc`): affected are the `rc-on` Long/Short text, both doc.go files, `orchcli/remotecontrol_test` and `paneoptions.go`. The `paneoptions.go` docs are not updated (finding 2).
- **RC now serves every agent through `remote_control_name`**: affected are `orchengine/template.yaml` lines 11–12, `orchengine/doc.go` line 117 and `config.go` line 68. The template's comments are not updated (finding 3).
- **New `cli.go` seam `markRemoteControl`**: its one wiring is `reed.MarkRemoteControlOn` at `cli.go:270`, and the tests stub it. Covered.
- **PATTERN entries**: `PATTERN-reed-panes` covers typed text and the wait mark, and `@lym_rc` is a pane option, so it does not conflict. No declared PATTERN changes to land. Covered.
- **Commit messages**: no `Pattern-Departure:` trailers. c48869a7's message matches the diff. Covered.

**Findings:**

1. **LOW, consistency** — `internal/slackcli/toggle.go:52`. The `slack bar` flag `--dir` still reads "run directory of the session whose bar is drawn". The bar now passes `@lym_orch_dir`, which is the session's orch directory and is empty for a task.
   - Evidence: `internal/slackcli/toggle.go:52` and `internal/reedengine/panebin.go:264`.
2. **LOW, consistency** — `internal/reedengine/paneoptions.go:4-5` and `:28`.
   - The file header says "No Go decision reads them back but the button click, which reads @lym_strand_name". The `strandNamePaneOption` doc says the same thing: a button click reads it to fill `{strand}`.
   - The RC click now also reads `@lym_strand_name`, through `clientAgentPaneVia`, to name the agent it acts on. The `reedengine/doc.go` update says so; these two comments contradict it.
   - Evidence: `paneoptions.go:4,28`, `internal/reedengine/remotecontrol.go:37`, and the `reedengine/doc.go` diff at line 170.
3. **LOW, consistency** — `internal/orchengine/template.yaml:11-12`.
   - `remote_control` says "the RC button on the orch session's status bar turns it on".
   - `remote_control_name` describes `{role}` as "the orch strand's role" and `{name}` as "its full name". It says the name is "also used by the RC button".
   - The RC button now belongs to whichever agent's pane is active, in any session, and the name is formed from that agent's strand name, not only the orch's. The template comments still describe an orch-only button.
   - Evidence: `template.yaml:11-12`, `internal/orchcli/remotecontrol.go:112`.

**Tool calls:**
1. Grep across .md, .go and .yaml for RC, `rc-on`, `slack bar`, `@lym_orch_dir`, `@lym_rc` and "remote control". The manifest held no docs, config templates or other readers of these outside the diff.
2. Grep of `pattern/PATTERN-reed-panes.md` and `PATTERN.md` for pane-option rules, and `orchengine/config.go` for the `RemoteControlName` doc. The manifest had not read these.
