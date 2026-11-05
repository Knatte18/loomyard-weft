---
round: 1
focus:
  - "Verbatim move: for each folded entry's background file removed or renamed in `pattern/`, check its text appears unchanged in the cluster's background file, and that rule lines dropped from `PATTERN.md` appear verbatim there. Question: is any rule lost, reworded or truncated?"
  - "Old-ID citations: grep every folded entry ID across `internal/`, tests, stencils, specs, skills, docs and heddle entries, excluding `docs/research` and history. Question: does any citation of a removed ID remain, or does any rewritten citation point at a cluster that does not hold that rule?"
  - "Overview size: measure `PATTERN.md` bytes against `MaxOverviewBytes` in `internal/pattern/check.go`. Question: is it at or below about 60% of the cap, with the cap and `HeadroomBytes` unchanged?"
  - "Section placement: compare `PATTERN.md` section headings before and after. Question: do all headings stay, and does each cluster sit under the section its subject belongs to?"
  - "Cluster coherence: read each new or renamed cluster entry (`PATTERN-fabric-boundary`, `PATTERN-code-conventions`, `PATTERN-pair-remote`, `PATTERN-reed-panes`, `PATTERN-lym-files`, `PATTERN-child-processes`). Question: does each rule line still state the rule's guard accurately, and does any entry's scope or enforcement pointer now misdescribe what the merged detail covers?"
  - "Code edits under `internal/`: read each non-pattern file in the diff. Question: is every change a pure ID rewrite in a comment or string, with no behavior change and no stale cross-reference to a path under `pattern/`?"
  - "Commit message and `_lym/landing/summary.md` claims: check them against the diff."
---

Seed focus from the decision record's Goal (the only answer key; no plan) and the one-commit diff stat.
