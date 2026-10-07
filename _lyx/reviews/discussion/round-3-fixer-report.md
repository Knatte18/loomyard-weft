# Round 3 fixer report

All four findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

## Changed

- **F1 (MEDIUM)**: D5 now bounds every agent-written part of the hold notice, each task label and the quoted message start: a fixed pair of delimiters, control characters and newlines replaced by a space, a 200-character cut, and at most five tasks named.
  It states why: a shell's label is its raw, possibly multi-line command, and `QueueNotice` refuses a line with a newline.
  An acceptance criterion pins a held turn end whose outstanding shell has a multi-line command: one notifier call, a single-line notice.
- **F2 (LOW)**: D5 fixes the line order, way forward before the tasks and the quote last, and says a delimiter character in the agent's text is replaced by a space, so no agent text stands outside the delimiters.
  The notice prefix labels every delimited part as the agent's words.
  The acceptance criterion pins the delimiters, the order and the no-delimiter-character rule.
  D8's stencil statement now covers the delimited text, task labels and message start.
- **F3 (LOW)**: D4's deadline sentence now reads "the caller's own `Spec.Timeout`, however that caller sets it", naming the `*_timeout_min` keys, a recipe row's `timeout_s`, a profile or `--timeout` value, with `run_timeout_min` only for a zero value.
- **F4 (NIT)**: the Auto-mode assumption names loom, webster, burler and batten-driven runs, matching D5.

## Deferred

Nothing deferred.
