# Round 5 fixer report

All seven findings fixed, in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.
`lyx loom validate-discussion` exits 0 after the edits.

## Changed

- **F1** — D13 now orders the preprocessing per pattern.
  Heredoc bodies drop first for all three patterns; the path pattern runs with quotes intact; the bare-word and spelling patterns run on the quote-blanked copy.
  The finding's matched text comes from the copy the pattern ran on.
- **F2** — D13 states that the matcher runs once per span and that every fenced block is scanned whatever its info string.
  The Bound adds the miss after an unterminated quote, and Open risks adds the false refusal for a fenced block that carries file content.
- **F3** — the `PATTERN-fabric-vocabulary` Constraint replaces its four-surface list with the scan's scope, names `WeftWorktree` as the `vocabExempt` helper tests may use, and states that a committed `testdata` fixture is not scanned.
- **F4** — D11 states that a run with an unreadable last state enters no profile line and is listed with the reason, and that a history with no round review files marks the run's findings not read, never zero.
  The per-profile acceptance criterion follows.
- **F5** — Scope Out now says the audit's location-free patterns are exposed to the plan check; D13 notes the patterns already live on the fabric side.
- **F6** — D13 and the `PATTERN-sole-parsers` Constraint state that the check reads the plan files' raw text inside `planparser`, by a model addition or a re-read, which the model does not hold for fenced blocks and inline spans.
- **F7** — the per-profile acceptance criterion states that a line says how many runs contribute findings.

## Deferred

None.
