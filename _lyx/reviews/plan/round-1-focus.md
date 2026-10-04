---
round: 1
focus:
  - "Card 3 (granularity): it edits about two dozen files and carries the model swap, label validation, kind rules, promote, claimability and the README render. Settle whether `labels.go` validation, `Promote` and the render/order change are independently reviewable units that belong in separate cards, or are inseparable because removing `Tier` and `Type` breaks every reader at once."
  - "Fidelity: walk each D1-D13 decision, each Constraint and each numbered Orchestrator decision in `_lyx/discussion/decision-record.md` to the card that carries it. Pay particular attention to the acceptance criteria on `--body-file -` refused when the payload also arrives on stdin, `upsert` of `issues` refusing duplicates, a fold never adding a label the target already carries, `into` accepting a done entry and a task, and `into` refused with `title`, `brief` or `labels`."
  - "Fidelity, scope side: find any card behavior (flags, payload keys, helpers, refusals) that neither a decision nor a constraint licenses, especially in cards 7, 8 and 9."
  - "Verify coverage: for every package a card targets (including `internal/boardengine/boardtest`, `internal/selfreportengine`, `plugins` docs and any package named only in a card's Edit list), settle whether the `## verify:` section runs its tests, tagged ones included."
  - "Attack-surface: for the new verbs and escape hatches (`intake close`, `intake import`, `--body-file`, a nil-vocabulary store, a path-only `Board` skipping label validation), settle whether each introducing card's `**Intent:**` states what it lets through and what bounds it."
  - "`ImpactSummary` on `Edit` and `Delete` cards: settle whether each states a real blast-radius conclusion rather than restating `Intent`."
  - "Card 4 and card 10: settle whether the migration rules (D12) and the D13 area-label list in the PR description are carried exactly as the decision record words them, with no extra mechanism added."
---

Seed focus only; each entry names where to look, none pre-states a verdict.
`Do not flag` items (derived dependency edges, `ImpactSummary` on non-`Edit`/`Delete` cards, gate-checked IDs) were excluded.
