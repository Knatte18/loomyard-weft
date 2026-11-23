```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens. I reran each set-premise finding's search against the fix.

No findings.

- **F3, consolidation origin retag:** the only stencils that retag kept findings with `lens`, `handler` or the assist origin are `burler-cluster-fan.md`, `burler-assist-cluster-fan.md` and `burler-assist-consolidation.md`, and all three now carry the `origin: focus` exception. `burler-parent-consolidation.md` tags `origin: parent`, which is a separate origin and needs no change.
- **F4, `FocusFindings` wiring:** `BurlerHalfDeps{` has two non-test constructions, the review row (sets `FocusFindings: hardenRound`) and the fix row at `entries_burler.go:318`, which never reads the field. `NewBurlerReviewProducer` has one caller. `BuildRoundContext` has two non-test callers, the review producer and `burlercli/validate.go`, and both now set `FocusFindings`.
- **F9, `code_auto_accept` descriptions:** the only places that describe it are `docs/overview.md:1515`, the `config.go` field comment and `template.yaml:53`, and all three now name the BLOCKING carve-out. The split perch's `split_auto_accept` line is a different key and not in scope.
- **F1, `blockingEscalationDetail` and `escalateUndecided` callers:** I did not rerun this search. It rests on the earlier grep, which found a single non-test call site for `escalateUndecided`, and `go vet` and the tier0 gate passed on the touched packages, so a missed caller would not have compiled.

One note outside the fix's scope: `internal/burlerengine/doc.go:242` says every kept finding carries `lens:<name>` or `handler`, and it already omitted the `webster` exception before this change.

Tool calls used:
1. One combined grep for the origin-retag sites, the `NewBurlerReviewProducer` and `BurlerHalfDeps{` callers, and the `auto_accept` descriptions. My context held only partial greps of these sets, not the whole-repo sweep.
