---
verdict: APPROVED
rationale: "All five round-1 findings are fixed in the current plan (card 12 keeps a re-begun batch's StartSHA per recoverbatch.go:207, card 9 names the stale comments, dirt shape and conditional no-bisector notice, card 10 cites D11), every D1-D11 decision and constraint is carried, and verify runs every targeted package untagged and integration-tagged; no new finding survives the rubric."
---

# Plan-Review round 1 — judge verdict

## The round-1 report

The report raised one BLOCKING, two LOW, and two NIT findings, and I agree with all five.
Its report carries no `## Focus departures` section, so there are no departures to ratify or reject.
Its focus settlements are sound, and I spot-checked them against the repo:

- `internal/landingshed/describe_test.go` writes its own stencil fixture and never pins the seeded Describe stencil's text, so card 10's conditional test update is moot rather than a missing target.
- `summaryparser` passes `summary.md`'s body through verbatim with no section validation, so the appended `## Integration suite triage` section cannot break the done-path summary parse.
- The callers each ImpactSummary claims match the code:
  `runVerifyCommand` is called only from `checkoutAndVerify` (`integration.go:166`);
  `AppendIntegrationFailure` is called from `integration.go:227` and `runlevel.go:1009`;
  `LocalizeIntegrationFailure` has one production caller, at `runlevel.go:986`;
  `ParseReport` is called from record-batch, recover-batch, and the integration stage.

## The fixes, checked against the current plan

- **F1 (baseline provenance):** card 12 makes `BeginBatch` keep a prior record's non-empty `StartSHA`.
  Today `beginbatch.go:396-411` records `head` unconditionally, and card 12 mirrors `recoverbatch.go:207-209`'s inheritance.
  Card 12 is licensed because D7's baseline, "the HEAD webster recorded before the first batch in execution order began", holds only if the recorded value survives a re-begin.
  It is a single, independently testable unit with `integration`-tagged tests.
  Its ImpactSummary names every reader, including `webstercli`'s `start_sha` envelope field, and no stencil consumes that field.
  Card 9's fixtures set batch 1's `StartSHA` directly, so appending card 12 after card 9 creates no build-order hazard.
- **F2:** card 9 now names the `RunResult.Outcome`/`StuckReason` doc comments (`runlevel.go:175-181`) and the call-site comment at `runlevel.go:700-705`.
- **F3:** card 9's dirty-tree case names the dirt: an untracked file, plus a tracked change identical at baseline and head.
  It routes a conflicting change to card 5's restore-failure case.
- **F4:** the no-bisector notice is limited to a regression, and the nil-bisector flaky case asserts that the flaky warning is its only warning.
- **F5:** card 10's Intent cites D11 and explains why the Describe rule is needed.

## Rubric sweep

- **Fidelity:** D1–D11 and every listed Constraint are carried by cards 1–12.
  The Describe stencil edit (card 10) and the `webster-spec.md` edit (card 11) are needed to deliver D11 and to keep the docs true, so neither adds unlicensed scope.
- **Verify coverage:** the targeted packages are `internal/websterengine`, `contracts/...`, and `internal/landingshed`, plus `internal/webstercli` as a reader of card 12's change.
  The verify runs all of them both untagged and `-tags integration`, and it vets `webstercli` under `smoke`.
- **Granularity, ImpactSummary, Custom:** no `Custom` card exists.
  Every `Edit` card's ImpactSummary states a caller-level conclusion rather than restating its Intent.
  The bundling in cards 3, 7, and 8 is justified as the report settles it.
