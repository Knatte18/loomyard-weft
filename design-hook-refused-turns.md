# Agents still try the forms lym's guards refuse, at a turn each

`hook-refused-turns` · Task · enhancement, process, claude · running · Code-Burler-Fix

## Seen (orch, 2026-11-01)

In the five heaviest Webster forks of the three latest tapestry runs, 6 turns were refused by lym's PreToolUse guards: four Python attempts and two raw `go build` or `go vet` runs.
Each refusal costs a turn at the fork's full context, and the fork then makes the call again in the allowed form.
The forks inherit Merriam's system prompt, which carries the guards' steering sentences, yet still reach for the refused forms first.

Python stays refused: it is no dependency of this Go repository, it is not on every machine, and agents use it to rewrite files by script; the guard follows the shuttle config's `python` key, so a hub for a Python repository turns it off.

## Idea (operator and orch, 2026-11-01)

- First read, in the refused turns, what the agent wanted Python or the raw `go` command for, such as editing a file or parsing JSON or text; note `edit-with-edit-tool` gives edits their way, and the steering names jq, grep and awk for parsing.
- Close the rest where it lies: the steering sentence's wording, the stencil's own instructions for building and vetting, or a guard that answers with the allowed command ready to run.
- Trial the same rule as a line in the rendered `CLAUDE.local.md` through heddle, in the same round as note `batching-in-claude-local`, since both are heddle text with no code change.
