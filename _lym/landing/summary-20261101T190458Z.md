# Agents may use subagents; headless claude launches are refused instead

`claudeengine` denied the in-process Agent tool in every run, or let only forks through in fork-mode runs, on the reasoning that all work must stay visible in the run's pane.
That kept a darn writer at large context from handing per-package work to cheaper helpers, and a Plan writer from handing lookups to a fresh subagent.
A subagent shows in the session's own pane and lym reads its transcript, so it hides nothing.
What does hide work is a headless `claude -p` launch, and that is what a guard now refuses.

## What changes

- `internal/shellguard/rows.go` gains a `headless-claude` row, installed in every run with no switch.
  It refuses a Bash launch of `claude` or `claude.exe` in command position with `-p` or `--print` among its arguments.
  Its steer and notice say the launch's work shows in no pane and name a subagent through the Agent tool as the way to hand work off.
  Its `Forms` join the no-denied-recovery scan.
- `internal/shuttleengine/claudeengine/settings.go` drops both Agent denies, their steers and notices, and the `allowAgentTool` input of `buildSettings` and `buildDenyNotice`.
  The webster fork guard (`steerWebsterForkDeny`) and the AskUserQuestion deny stay.
- `shuttleengine.Spec.AllowAgentTool` goes, and with it the orch spec's setting of it.
- `claude.yaml`'s `deny_agent_tool` key goes from the template and `Config`.
  It is declared a retired key, so a layer that still holds it, such as a hub that set it to `false` as a stopgap, loads without it, and a local override setting it is refused with the way forward of deleting it.
- The darn writer stencil tells the writer to hand lookups and decided per-package edits to fresh subagents (`subagent_type: "general-purpose"`, `model: "sonnet"`), one per package with its own files, launched together in one message, each given its files, its edits and the gate rule with its Bash tool timeout.
  The design, cross-package changes, commits and the change description stay in the session.
  The Plan writer stencil gives the same advice for batches of lookups.

## Tests

- `rows_integration_test.go` pins which launches the new row refuses and which it lets through, among them an interactive launch, `claude` as an argument, a quoted mention, `claudette` and a `-p` of a later command.
- The claudeengine settings tests now assert that no input installs an Agent entry and that the headless claude deny is installed under every input.
  `settings-golden.json` is regenerated over the four remaining deny inputs.
- `config_test.go` pins that a repo default still holding `deny_agent_tool` loads.
- The live smoke tests change with the rule:
  `TestSmokeGuardrailDeniesHeadlessClaude` replaces the Agent-deny smoke test,
  and the burler rogue-fork smoke test keeps only its audit assertion, since no hook now refuses an Agent call inside a fork.

## Docs

The `shellguard`, `claudeengine`, `shuttleengine` and `burlerengine` package docs, `docs/overview.md`'s shuttle entry and `pattern/PATTERN-interactive-spawn.md` describe the new rule.

## Design decisions

- The guard is a `shellguard` row, not a claudeengine-only hook, because `PATTERN-shell-guard-table` makes that table the only place a shell command guard is declared.
- `deny_agent_tool` is retired through `configengine`'s retired-key mechanism, not just removed, so hubs that still carry the key keep loading.
  The `shuttle.claude_deny_agent_tool` key move stays, so an old shuttle layer's value lands under the retired key and is dropped on load.
- The steer and notice say "print mode" and spell neither flag after the word claude, so the `PATTERN-interactive-spawn` scan of production string literals does not flag them.
