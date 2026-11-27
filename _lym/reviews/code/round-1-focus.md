---
round: 1
focus:
  - "Transient classes against the new wording: `ClassifyHalt` and `failedHalt` treat every non-empty `Status.Transient` alike, including `agent-start` and `host-resources`. Check `anomaly.go`, `internal/loomcli/doc.go`, the reflection stencil, `_lym/landing/summary.md` and the commit messages against what `decideDrive`, `networkFailure` and the loop's own re-step do for each class: is each claim about a network or host failure and about driver re-steps true for all of them?"
  - "The amended entry's 'until the driver's re-steps are spent': trace a transient failed step through the halt note, the re-steps and the final hand-back or success. Settle whether the recorded state and note meet what the entry asks, both for a stop a re-step clears and for one that outlasts every re-step."
  - "The reflection template's `transient-stop` rule: check whether the notes and trace the reflection agent receives can show that re-steps at one row never cleared the halt, or that no retry could fix it, and whether a persistent failure can be silently dropped."
  - "Other consumers of the anomaly kind and of `failed` halts, such as the halt-note readers in `internal/reportengine`, `internal/haltnote` and any spec or doc that lists the kinds: does the new `transient-stop` value need a matching change that the diff lacks?"
  - "Parent lock wait logging in `parentguard.go`: check the start, holder-change and end lines on every exit of `Acquire` (grant, pause, unreadable record, cancellation, other error), the wall-clock against awake-time arithmetic and the comment's claim about suspension on each platform the repository supports, and whether the added behaviour needs coverage."
  - "Change description `## Merge danger`: check the `two-way` door and the blast radius `loom` against everything the diff touches, including the landing path and the reflection stencil."
---

Seed focus drawn from the decision record's `## Goal` entry, the diff over the review range and the change description.
None of the entries is a verdict.
