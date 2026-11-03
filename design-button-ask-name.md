# A status-bar button asks for a value, so an agent opened from it gets a name

`button-ask-name` · Task · enhancement, reed, config · done

## Seen (operator, 2026-11-01)

The reed status bar's shipped `agent` button runs `lym shuttle open --role agent --model 'opus[high]'`, so every agent opened from it gets the strand name `agent`, and a second one only reed's number, `agent-2`.
With several agents each looking at its own thing, nothing in the strand name says which is which.

## Design (operator and orch, 2026-11-01)

- A button entry in `reed.yaml` (and `desktop.yaml`'s machine buttons) takes an optional `ask`, a prompt text beside `confirm`.
- A click on an entry with `ask` raises tmux's `command-prompt` on the clicking client with that text; the answer fills the placeholder `{answer}` in the entry's `run`, shell-quoted like the other placeholders, and the entry then runs as today, through `confirm` when it has one.
- An empty answer fills `{answer}` with the entry's `ask_default`, so a button still runs when the operator just presses Enter; an Escape cancels the click.
- `{answer}` is valid only in an entry with `ask`, checked where the other button rules are checked.
- The shipped `agent` buttons, in the POSIX and Windows templates, become `ask: "Agent name:"`, `ask_default: agent` and `run` with `--role {answer}`.
- The shipped `term` buttons are labelled `terminal` (operator, 2026-11-01).
