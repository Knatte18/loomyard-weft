# reed: a status-bar button asks for a value that fills {answer}

The shipped `agent` button ran `lym shuttle open --role agent`, so every agent opened from the status bar got the strand name `agent`, and later ones only reed's number.
With several agents open, the strand name did not say which was which.

A button entry in `reed.yaml` and in `desktop.yaml`'s machine buttons now takes an optional `ask`, a prompt text beside `confirm`, and an optional `ask_default`.
Clicking an entry with `ask` raises tmux's `command-prompt` on the clicking client.
The answer fills `{answer}` in the entry's `run` and `confirm`, quoted like the other placeholders, and the entry then runs as before, through `confirm` when it has one.
An empty answer takes `ask_default`, and Escape cancels the click.

The shipped `agent` buttons in the POSIX and Windows templates now use `ask: "Agent name:"`, `ask_default: agent` and `--role {answer}`.
The shipped `term` buttons are labelled `terminal`.

## Design decisions

- `{answer}` is valid only in an entry with `ask`, and `ask_default` needs an `ask`; `buttonEntryProblem` refuses both otherwise, where the other button rules are checked.
- The ask text may not hold a comma, because tmux splits a `command-prompt` prompt at its commas.
  A static text with a comma is refused at validation.
  A text that gets a comma from a filled placeholder such as `{slug}` is left out of the row with a Warn at render, and shows a message on the clicking client if clicked from a stale spec.
- The answer travels through the session user option `@lym_button_answer`, not through the re-entry command line.
  tmux can escape the answer for its own parser but cannot quote it for a shell, so the prompt template stores it with `set-option` and the click reads it back.
  The answer is trimmed of surrounding space.
- `lym reed button` gains `--answered`, `--answer` flags.
  `--answered` needs `--hash`, like `--confirmed`, and a re-entered click runs only when the entry's run text still hashes to the prompt's hash.
  When an entry has both `ask` and `confirm`, the ask comes first and the confirmed re-entry carries the answer in `--answer`.
- `composeConfirmedButtonLine` became `composeButtonReentryLine`, taking the stage flag and the values to carry; the `{strand}` and `{answer}` names are now constants, and `buttonEntry.names` replaces the repeated `strings.Contains` checks.
- `tmuxQuote` is split so the ask prompt's `set-option` argument, which tmux does not format-expand, is escaped without doubling `#`.

## To check by hand

- The ask path in a live tmux: click the shipped `agent` button, type a name, press Enter, then repeat with Enter alone and with Escape.
  A review left `ask-path-no-real-tmux-test` (MEDIUM) open; the integration tests in `internal/reedengine/buttonclick_integration_test.go` now drive the prompt through a real tmux client, but no review has seen that result.
