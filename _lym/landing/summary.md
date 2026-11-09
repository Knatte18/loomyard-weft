# Reed ask buttons undo the backslash tmux keeps before $NAME

An ask button's answer reached the run text with an extra backslash before a `$` that starts a variable name, as in `\$HOME` instead of `$HOME`.
The adopt check of root failed on this at tier 2, in `TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact`.

## Cause

The `command-prompt` template stores the answer through tmux's `%%%`, which escapes every `$` of the answer as `\$`.
Some tmux builds, 3.4 among them, keep that backslash when a variable name follows, so the stored option holds `\$HOME` where other builds hold `$HOME`.
The fault is in the code, not the machine: the answer path assumed one tmux behaviour.

## Change

In `internal/reedengine/buttonclick.go`:

- The template now stores the answer behind an escaped marker, `\$A%%%`.
  The marker goes through the same tmux escaping as the answer, so the stored marker shows whether this server kept the backslash.
- `decodeStoredAnswer` reads the marker back.
  A stored `\$A` means the server keeps the backslash, and the decoder drops every backslash directly before a `$` that starts a variable name (`[A-Za-z_{]`).
  A stored `$A` means it does not, and the answer is used as stored.
- `buttonAnswer` decodes before it trims and applies the ask default.

`internal/reedengine/doc.go` describes the marker in the ask-button flow.

## Design notes

- The marker is the way to tell the two behaviours apart without asking for the tmux version.
  The server's behaviour itself decides, so no version table is needed.
- In the keeping mode, tmux also escapes each backslash the answer holds, so a kept escape backslash always stands directly before the `$`.
  A typed `\$HOME` therefore comes back as `\$HOME`, and the decoder never eats a typed backslash.
- A `$` followed by anything other than a name start, such as `$-`, loses no backslash, because tmux does not keep one there.

## Tests

`TestRunButtonVia` gains two cases for the keeping mode, one where the server unescapes every `$` and one where it keeps the escapes, and the existing ask cases carry the marker.
The unit tests ran in the task.
Run `TestRunButton_RealTmux` on a machine with tmux 3.4 to confirm the integration case passes there.
