---
verdict: APPROVED
rationale: "All seven round-1 findings are fixed in the live cards: card 11 moves the entry observation into observeStepEntry at the end of loomPreStepUnmarked (matching shedverbs/step.go, where a PreStep or BuildShed refusal never reaches AfterStep) and adds the two-step handoff test, card 10 lists the cancelled-context, unreadable-status and undecodable-product arms and edits selfreport_github_test.go, both bounds name their residuals, and the halt-note line and stencil sentence now match verbatim; every PR-Gate F1/F2 directive and decision-record addition (a)/(b) is carried, and go test ./... plus -tags integration cover loomcli, loomengine and contracts/stencils."
---

# Plan review, round 1 — judge verdict

The answer key is the decision record plus `_lyx/loom/rework/round-1/findings.md`: round 1's `record.json` carries `class: required` and `first_card: 10`, which matches the plan's frontmatter.

## The report's findings against the current plan

The report approved, and its fixer has since rewritten cards 10, 11 and 12.
I judged the cards as they now stand.

- **F1 (entry observation placement).**
  Card 11 now adds `observeStepEntry()` and calls it last in `loomPreStepUnmarked`, after the busy probe, the bootstrap and `reed.Up()`.
  `internal/shedverbs/step.go:159-201` confirms the premise: a `PreStep` or `BuildShed` refusal returns before `AfterStep`, and `PostStep` runs only after a successful `shed.Step`.
  The observation still runs before `shed.Step`, so the handoff marker `loomPostStep` writes is consumed by the next step's observation, as F1 of the findings requires.
- **F2 (no real-sequence test).**
  Card 11 adds a `halt_test.go` case that drives the generic step body twice, with `PreStep` calling `observeStepEntry()` and the status seeded once.
  This works around `stepEnvelopeOver` (`internal/loomcli/halt_test.go:328`), which clears `PreStep` and reseeds the status on every call.
- **F3 and F6 (unnamed residuals).**
  Card 11's bound names both residuals: the `crash-resume` after a `BuildShed` refusal or a `shed.Step` error that leaves the run `running`, and a halt whose Tier 1 filing failed or was skipped.
  Card 12 restates the halt residual.
- **F4 (missing test arms).**
  Card 10 adds the unreadable-status, undecodable-product, cancelled-with-no-crash and cancelled-with-crash cases.
- **F5 (a test doc goes false).**
  Card 10's Edit list now includes `internal/loomcli/selfreport_github_test.go`.
  The test at line 152 gets a new doc and an assertion on the pending entry.
- **F7 (wording).**
  The note line and the stencil sentence now share `Tier 1 anomaly filing files the halt event itself`.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.

## My own pass over the rubric

- **Fidelity.**
  Addition (a) is carried by card 11, with the existing skips kept, and by cards 11 and 12 for not double-filing.
  Addition (b) is carried by card 10.
  Every directive in findings F1 and F2 is carried: the skips, the placement after the busy probe, the ownership statement, tests through `loomCLI.fileIssue`, the pending list in the filed-title marker, and the resumed-run test.
  Two items go beyond those directives: the `testing.Testing()` refusing filer is the instrument behind "tests through `loomCLI.fileIssue`", and the `docs/overview.md` edit is licensed by the decision record's same-commit docs constraint.
- **Attack surface.**
  Each new path states its bound in its own card:
  - card 10: the pending retry;
  - card 11: the `fileIssue` seam, assigned only in `newLoomCLI`, and the new public filing under `step`;
  - card 12: the reflection leaving the halt event to Tier 1.
- **Verify coverage.**
  `go test ./...` and `go test -tags integration ./...` run every package the cards target.
- **Granularity, `ImpactSummary`, `Custom`.**
  There are three cohesive cards, and no `Custom` card.
  Each `Edit` card's `ImpactSummary` states a real blast radius, not a restatement of its Intent.

One implementation note, not a finding: `haltFixture` and other hand-built receivers set no `fileIssue`.
So any test that turns the knob on and reaches a filing pass must set the fake, as card 11 already says the Tier 1 cases do.
