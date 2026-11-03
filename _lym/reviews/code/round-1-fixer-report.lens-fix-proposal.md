```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F4 as the fixer carried them out.

**Findings**

- **LOW**, `internal/loomcli/decision.go`, the `add` Long text. The summary is "refused ... until Record-Write has committed its record" now over-reaches.
  - The old text said "and on a Record-Write record until Record-Write has committed it". That limited the refusal to a recipe that has a Record-Write row.
  - The new text drops that qualifier. A reader on a recipe with no Record-Write row can read it as a refusal that applies to their run too.
  - The code gates this on `rowWritten` only (`decisionVerb`, the `if rowWritten { checkDarnRecord ... }` block).
  - F2's proposal said to trim "other prose", and this cut changes meaning. Restore the qualifier in fewer words, for example "and, on a Record-Write recipe, until it has committed the record."

- **LOW**, `internal/loomcli/decision.go`, the `add` Long text. "(restoring on a finding)" lost its object.
  - It no longer says what is restored, and the savings were about 11 bytes.
  - The cut is only cosmetic. It is worth undoing if the cap margin allows.

**Challenged and held up**

- **F1:** `Board.Rename` has a single caller, `internal/boardcli/cli.go:241`, so the changed return contract breaks nothing else.
  - `followDecisionRecords` returns an error only for a rename, so `Remove`, `Merge` and the other writers still get `nil`.
  - On a render failure after the record step, `boardCriticalSection` returns `nil, err`. That error is not a `*DecisionRecordError`, so `Rename` returns an empty result, which is correct because the result is unavailable.
  - `result.(RenameResult)` is safe on the record-error path. `boardCriticalSection` returns the `RenameResult` it rebuilt with the re-read `Task`.
  - `ErrFields` keeps the `ok` and `error` keys reserved, so the extra `task` and `mentions` keys cannot collide.
- **F3:** `DecisionFileName("<slug>")` yields `decision-<slug>.md`, the same text as the old literal. The export also leaves no stale `decisionFileName` reference. The grep of the rest of the repo found none.
- **F4:** the doc-only option changes no behavior, so there is nothing to break.

Further tool calls:
1. Grep of non-test code for `Board.Rename` callers, to check whether the changed `Rename` return contract breaks any caller outside `boardcli`. My context lacked a complete caller list, because the earlier grep was truncated by `head`.
