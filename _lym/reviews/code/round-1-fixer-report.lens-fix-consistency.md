```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `session_test.go` change set (three new rows, `seeded` field, `PromptOffset` wiring) for agreement with its surroundings.

**Findings**

- LOW, doc comment of `TestRunner_SessionIdle_ReturnsScriptedClassification` (`session_test.go:231-234`).
  - The comment describes each row as "a series of probes ... each appending events", and lists the behaviors the rows pin.
  - It names no row that starts from a seeded events file with a non-zero prompt offset, which the new `seeded` field and rows add. A reader of the comment alone would not learn that rows can fix the offset, or that the readiness fallback to the whole-file fold is pinned here.
  - Evidence: the new `seeded` field comment says "its length is the run's prompt offset", but the function doc still lists only "ready, held or unknown readiness reading and its fallback to the pane...". The phrase "fallback to the pane" now has a twin, the fallback to the whole-file fold, that the comment leaves out.
  - Proposed change: add a clause such as "a prompt offset past the events a run has already written, whose last turn end the readiness reading then takes".

No other disagreement found.
- The `unreadable` and `seeded` field comments are consistent with each other.
- The new rows' reasons and probes match the strings used by the existing "a running turn is held" and `idleAtEnd` rows.
- `PromptOffset` is `int64`, matching the `int64(len(...))` conversion.
- The zero-offset row leaves `seeded` empty, so it stays equivalent to the pre-change setup.

**Further tool calls:** none. The context already held the changed file, the review and the rubric.
