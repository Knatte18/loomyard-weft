# Card 4 — claude-bash-stdin-hook

**Edit:**
- `internal/shuttleengine/claudeengine#buildSettings`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Create:**
- `internal/shuttleengine/claudeengine/testmain_test.go`
- `internal/shuttleengine/claudeengine/bashstdin_integration_test.go`
- `internal/shuttlecli/smoke_bashstdin_test.go`

**Intent:**
D3: every Bash command a lyx-spawned agent runs gets `/dev/null` as its default stdin, so an interpreter left reading the tool's open stdin (the `board-model` run's `python3 -` whose heredoc was attached to the next command) ends at once instead of hanging the session.

The primary route applies; the fallback is not selected.
Claude Code's hook contract (code.claude.com/docs/en/hooks, "PreToolUse decision control") documents `hookSpecificOutput.updatedInput` as "Only allowed when `permissionDecision` is `allow` or omitted", and an omitted `permissionDecision` "doesn't decide; the normal permission flow applies".
The hook therefore returns `updatedInput` with no `permissionDecision`, so it grants nothing and the agents' allow and deny rules still decide the call; the smoke test below pins that the matching is unchanged.

`buildSettings` installs, for every run (autonomous and interactive, fork mode or not), one more `PreToolUse` entry with matcher `Bash` whose shell command:
reads the payload, extracts `tool_input.command` as its JSON-encoded string,
and, unless that string already starts with the prefix, echoes `{"hookSpecificOutput":{"hookEventName":"PreToolUse","updatedInput":{"command":"<prefix><command>"}}}`, reusing the JSON-encoded command verbatim so no re-escaping happens.
The prefix is one named constant, `exec </dev/null; `.
A command that already carries it, or a payload with no command, produces no output, which leaves the call unchanged.
The hook exits 0 in every case and denies nothing.
It is a separate entry from the fork-mode webster-verb guard, which keeps its own behaviour.
A heredoc, pipe or explicit redirect attached to a command still supplies that command's stdin, since only the shell's default stdin changes.
Bound: the hook rewrites only the default stdin; it denies nothing and changes no other part of the command.

`testmain_test.go` gives `claudeengine` its `TestMain` through `tmuxkit.Main`, since the package gains its first `integration` file (PATTERN-tmux-test-isolation).

Tests:
- `settings_test.go`: every run mode's settings carry the Bash rewrite entry, and its command embeds the prefix constant.
- `bashstdin_integration_test.go` runs the composed hook command under `sh` with real payloads:
  a plain command comes back with the prefix and decodes as JSON with no `permissionDecision`;
  a command containing quotes, backslashes and newlines round-trips byte-exact after the prefix;
  feeding the hook's own output command back in produces no output (no double prefix);
  the rewritten `cat -` exits at once with empty output while the test holds the shell's stdin open;
  the rewritten `cat - <<'EOF'` heredoc command still prints its heredoc body.
- `smoke_bashstdin_test.go` (`smoke` tag, following `smoke_guardrail_test.go`): a real agent in a hubforge hub with one dirty tracked file is told to run `git status` and `git reset --hard HEAD`;
  the run reaches done with `git status`'s output written, and the dirty file is still dirty, so an allowed command stays allowed and a denied one stays denied with the prefix applied.
  It skips when the operator's Claude settings carry no `git reset --hard` deny rule.

Document the rewrite and why it grants nothing in `claudeengine/doc.go` and in `settings.go`'s file comment.

**ImpactSummary:** every lyx-spawned agent's Bash calls gain a stdin-closing prefix through one new hook entry; existing hook entries are untouched.
