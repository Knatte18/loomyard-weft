---
round: 1
focus:
  - "Granularity across cards 3 and 6: both edit internal/loomshed#NewPlanDirRotator and internal/loomshed/planwrite_test.go, with card 3 changing the closure to return an always-empty amendment and card 6 filling it in. Settle whether this split is one independently reviewable unit per card, or whether the rotator's shape change belongs wholly in one card. Judge granularity only; an incomplete dependency list between the two is not a finding."
  - "Decision-record fidelity, D1 to D7 plus every Constraint and Acceptance criterion in _lyx/discussion/decision-record.md: confirm each one is carried by some card. Pay particular attention to D5's 'embedded and seeded' and the Constraint that the stencil enumeration tests must include the new stencil (card 4 relies on registry_test.go and the append-after-loom-rubric-webster-review ordering), and to the Acceptance criterion that a command built on each supported shell quotes the notice as one argument (card 2 names posix and pwsh). Also confirm no card adds scope the record does not license."
  - "Factual accuracy of each ImpactSummary: card 6's claim that every loom wiring already sets Env.StencilsDir, so the new requireAbsRoot check breaks no existing planWriteEntry construction, production or test; card 3's claim that loomshed.NewPlanDirRotator is the preparation seam's only non-nil supplier; card 2's claim that Claude.Prepare and the package's own tests are the builders' only callers. Settle whether each summary is a real blast-radius conclusion that matches the tree."
  - "Card 1's instruction to keep the Agent and AskUserQuestion install conditions in one place that both buildSettings and buildDenyNotice read. This reshapes buildSettings' body, which the card lists under Uses rather than as a symbol target (settings.go is a file-level Edit target). Settle whether the card's intent, tests and ImpactSummary ('buildSettings' output is unchanged') describe that refactor adequately."
---

The plan has six cards over four packages plus one stencil.
The mechanical gate already covers format, glyph resolution, field overlap and ImpactSummary presence, so none of these entries asks for those checks.
Verify coverage looks sound on a first read: `go test -tags integration ./...` runs every touched package, and `go vet -tags smoke ./...` only compiles the live-substrate smoke files, which is not a finding.
Card 5 is Create-only and carries no ImpactSummary, which the per-type table permits.
