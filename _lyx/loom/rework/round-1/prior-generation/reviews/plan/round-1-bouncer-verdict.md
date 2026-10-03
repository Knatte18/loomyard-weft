---
verdict: BLOCKING
rationale: "Cards 7 (Tier 1 anomaly filing under step) and 8 (pending-retry marker) still stand after the fixer pass, and the decision record, the sole answer key with no rework round, puts Tier 1 filing under Scope Out as run-only and licenses no pending-retry; F3 to F8 are fixed in the current plan"
---

# Plan-Review judge, round 1

The review's findings are all legitimate under the rubric, and I ratify each one.
The report has no `## Focus departures` section, so there are no departures to rule on.
The fixer had already edited the plan before this pass, so I judged the plan as it stands now.

## Still open: F1 and F2, BLOCKING

`_lyx/loom/rework/` does not exist, so `_lyx/discussion/decision-record.md` is the whole answer key.
Its Scope Out reads "Tier 1 anomaly filing (`detectAndFileAnomalies`) stays `run`-only."
Card 7 does the opposite, and card 8 changes the filed-title marker and how a public-issue filer retries.
Nothing in the record mentions either change.
Several other parts of the plan exist only because card 7 does:

- the overview's summary line, its "operator additions", "resumed-halt loss" and "who files a halt" sections, and its mention of `fileIssue` in "one fakeable reflection seam";
- card 2's Intent ("file Tier 1 anomalies");
- card 6's `tierOneOn` ownership line, its test, and its note that card 7 extends the scenario;
- card 10's rule about a halt note that says Tier 1 files the halt;
- card 11's CONSTRAINTS clause on Tier 1 under step, and its selfreport module line.

The fixer deferred F1 and F2 as an operator decision.
That is a reasonable thing to raise, but it does not make the plan fit its answer key.
The overview's "operator additions" section is plan text, and the plan cannot license its own scope.
There are two ways to resolve this:

- The plan drops cards 7 and 8 and everything that depends on them, and adds a Shared Decision recording that the board's post-Discussion addition is not carried, so the hub can take it back to Discussion.
  This is within Plan-Write's write surface.
- The decision record is amended through a `goto` to Discussion so that it licenses the addition.

Either way, until one of these happens the plan fails the fidelity rule.

## Resolved: F3 to F8

- **F3:** the overview's decision "a failed halt's error envelope carries the friction status" settles this.
  Card 2 puts `AfterStep`'s return on the error envelope of every `shed.Step` error and adds `friction` to the closed key set of every error envelope.
  It also merges `PostRun` extras onto `run`'s error envelope, with `battenPostRun` returning nil on a non-nil `runErr`.
  Card 6's `failed` arms now return their status, and a scenario tests the step error envelope.
  I checked the premise: `shedengine` returns the producer error and the unrecognised-outcome error unwrapped, and persists the same `err.Error()` (`internal/shedengine/run.go`).
  So `failedHalt`'s equality test does match the error and the persisted status.
  `./internal/battencli/...` is now in the verify section.
- **F4:** card 6 gained a "What this lets through, and the bound" paragraph.
  It names the `friction_timeout_min` delay, the held-lock skip, filing only through `lyx selfreport create` to `Knatte18/loomyard`, and card 10's content rule.
- **F5:** card 1 now limits one `Reflect` call to one `deps.Timeout`, measured through `deps.Clock`.
  A spawn after an attach wait gets only the remaining budget, and with no budget left nothing spawns.
  Both cases are tested.
  This fits `Runner.Attach`, which sets a fresh `now+Timeout` deadline (`internal/shuttleengine/attach.go`).
- **F6:** card 1 now covers each failure the record adds: a failed read, a failed write, and a failed spec build on the settle path.
  Each returns `StatusFailed` with a nil error, and an unparsable record is discarded.
  Tests cover both the unparsable and the unwritable case.
- **F7:** `halt_test.go` now tests that a `done` result after the row recorded `failed` reports `failed`.
  The scenario also states that the fake shuttle stands in for the agent's filing and that the `selfreport` knob is off.
- **F8:** the sentence in card 9 about the stop report is gone.

## New, LOW: card 2's ImpactSummary names the wrong consumer

Card 2's ImpactSummary says "lifecycle leaves the hook nil and only gains an empty `friction` key".
No lifecycle module builds a `shedverbs.Spec`.
The modules that do are `battencli`, whose step envelope gains an empty `friction` key, and `shedcli`, which delegates to the recipe module's own `Arm`.
On its own this would not block, but the conclusion should name the modules it is about.
