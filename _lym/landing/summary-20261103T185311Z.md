# Status-bar buttons ask for a value; the agent button asks for its name

The reed status bar's `agent` button opened every agent under the strand name `agent`, and a second one only as `agent-2`.
With several agents open, the strand names did not say which agent was which.
A button entry can now ask the operator for a value before it runs.
The shipped `agent` buttons use this to ask for the strand name.

## What changes

- A button entry in `reed.yaml`, or in `desktop.yaml`'s machine buttons, takes an optional `ask` (a prompt text) and an optional `ask_default`.
- A click on an entry with `ask` raises tmux's `command-prompt` on the clicking client.
  The answer fills `{answer}` in the entry's `run`, shell-quoted like the other placeholders.
  An empty answer takes `ask_default`, and Escape cancels the click.
  When the entry also has `confirm`, the confirm prompt follows the ask, and `{answer}` can appear in its text.
- `{answer}` and `ask_default` are valid only in an entry with `ask`.
  The ask text follows the confirm text's rules, no `#`, no `%` and no control character, and it holds no comma, since tmux splits a prompt at its commas.
  An entry that breaks a rule is skipped with a Warn, where the other button rules are checked (`buttonEntryProblem`).
- The shipped `agent` buttons in the POSIX and Windows templates are now `ask: "Agent name:"` and `ask_default: agent`, and their `run` passes `--role {answer}`.
- The shipped `term` buttons are now labelled `terminal`.

## How the answer reaches the run

`command-prompt` can escape an answer for tmux's own parser with `%%%`, but it cannot quote it for a shell.
So the prompt's template runs `set-option` to store the answer in a new session option, `@lym_button_answer`, and then a `run-shell` line that calls `lym reed button` again with `--answered` and the hash of the entry's run text.
The answered call reads that option and fills `{answer}` with the shell quoting the other placeholders use.
Like the confirmed path, it runs the entry only when the run text still hashes to the hash the prompt was raised for.
A confirm prompt after the ask carries the answer on its own line as `--answer`, which is already shell-quoted.
A `{strand}` entry carries the strand name through both prompts, so the answered call does not read the pane again.

The old `composeConfirmedButtonLine` is now `composeButtonReentryLine`, which takes the stage flag (`--answered` or `--confirmed`) and appends `--strand` and `--answer` when the click has filled them.
`tmuxQuote` is now built on a new `tmuxString`, which escapes for tmux's parser without doubling `#`.
The `set-option` target uses `tmuxString`, since tmux does not format-expand that target.
`fillConfirmPlaceholders` is renamed `fillPromptPlaceholders`, because it now fills the ask text too.

## Design decisions

- The answer travels through a session option rather than on the `run-shell` line.
  `%%%` escapes only for tmux's double-quoted strings, and a shell-quoted word cannot be built inside tmux's template.
  A manual check on a private tmux 3.6 server confirmed that a double-quoted `set-option` value holding the escapes `%%%` produces (`\;`, `\~`, `\$`, `\"`, `\\`) decodes to the literal text, with `#{…}` left unexpanded.
- Commas are refused in the ask text rather than passed with `command-prompt -l`.
  psmux on Windows has no tmux version floor, so the `-l` flag cannot be relied on there.
- The shipped `agent` button does not validate the name.
  An answer that is not a valid role name makes `lym shuttle open` fail, and the click shows that failure as `lym: agent failed (exit N): …`, the same as any failing button.

## Tests

- `TestRunButtonVia` gains rows for each new path:
  - the ask prompt's exact `command-prompt` argv;
  - an answered run with the stored answer trimmed and quoted, and with an empty answer taking `ask_default`;
  - a failed read of the answer;
  - an answered call whose hash no longer matches;
  - an answered `{strand}` entry that uses the carried name;
  - a confirm prompt that carries the answer;
  - a confirmed call that runs with the carried answer and reads no option.
- The re-entry line test covers both stages and the `--answer` word in both shell dialects, and pins `tmuxString`.
- `TestPlaceButtons` adds a valid ask entry and invalid rows for `#`, `%`, a tab and a comma in the ask text, `{strand}` in the ask text without `window: agents`, `{answer}` without `ask`, and `ask_default` without `ask`.
- `TestPinButtonRowLocked` pins that the rendered spec holds the filled ask text and the default, and leaves `{answer}` for the click.
- The shipped-template test pins the new `agent` and `terminal` entries in both templates.
- The new `TestValidateButtonFlags` pins that `--answered` without `--hash` is refused.
