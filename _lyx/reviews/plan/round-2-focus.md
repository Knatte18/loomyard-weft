---
round: 2
focus:
  - "Marker coexistence: the plan puts a `//lyx:guard` marker on the line directly above a test's `func Test…` line, while `cmd/testtiming/keep.go` (`scanKeeps`) accepts a `//testtiming:keep` marker only directly above that line, and several guarded targets already carry a keep marker. Settle whether the plan states one rule under which both markers can sit on one test, covers both parsers and their tests, and keeps `cmd/testtiming` and the impact-set derivation both passing."
  - "Card 11 premise: the tmux-tier card asks for a change to the dot-fill harness window size. Settle whether the card's stated cause matches the current code of `newDotFillHarnessOn` and its callers in `internal/reedcli`, and whether the card gives an implementer a reproducible route to the failure `docs/benchmarks/test-suite-timing.md` records."
---

Both entries restate the sites they depend on; neither caps severity or pre-states a verdict.
