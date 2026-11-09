# Reed ask buttons keep a $NAME in a typed answer intact on tmux 3.4

The adopt check of root failed at tier 2 on osl-1046 in `TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact`:
an answer holding `$HOME` reached `{answer}` as `\$HOME`.
This is a code failure, not a machine failure.
The machine runs Ubuntu's tmux 3.4, which is above reed's pinned minimum of 3.3.

## Cause

An ask entry's `command-prompt` template stores the answer through tmux's `%%%`,
which escapes every `$` of the answer as `\$` for tmux's own command parser.
The tmux 3.4 parser drops that backslash when a non-name character follows the `$`,
but keeps it when a variable name follows, `[A-Za-z_{]`.
So `"\$HOME"` parses to `\$HOME` while `"\$-"` parses to `$-`.
The same parse shows up in `set-option`, `display-message` and `set-environment`,
so it comes from the parser, not from the option store.

## Fix

`internal/reedengine/buttonclick.go`:

- The ask template stores an escaped marker, `\$A`, directly before `%%%`.
  The marker passes through the same parse as the answer,
  so the stored value shows whether this server kept the backslash.
- `decodeStoredAnswer` strips the marker.
  When the marker came back as `\$A`, it also drops the backslash before each `$` that starts a variable name.
  tmux escapes each backslash the user typed as well,
  so the backslash directly before such a `$` is always one tmux kept, never one the user typed.
  For example, a typed `\$HOME` is stored as `\\$HOME` and decodes back to `\$HOME`.
- When the marker came back as `$A`, from a tmux that unescapes every `\$`, the answer is used as stored.

## Tests and docs

- `buttonclick_test.go`: the expected ask template carries the marker.
  The answered-call rows read stored values with the marker in front.
  Two new rows cover the decode for each parser behavior.
- The real-tmux integration row that failed is unchanged and pins the fix end to end.
- `doc.go` describes the marker and the decode.
