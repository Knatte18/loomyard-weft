---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/07-anomalies-under-step.md, _lyx/plan/08-anomaly-pending-retry.md, _lyx/plan/00-overview.md:11
    summary: Cards 7 and 8 and the overview's operator-additions, resumed-halt-loss and who-files-a-halt sections carry Tier 1 under step and a pending-retry, scope the decision record puts Out and never licenses
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/06-halt-notes.md:32, _lyx/plan/02-step-after-hook.md:22, _lyx/plan/10-reflection-stencil.md:26, _lyx/plan/11-friction-capture-docs.md:15
    summary: The Tier 1 addition is threaded through cards 2, 6, 10 and 11 (Intent, tierOneOn ownership line and its test, scenario extension, stencil rule, CONSTRAINTS clause, selfreport module line), so dropping cards 7 and 8 alone leaves unlicensed scope behind
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/02-step-after-hook.md:54
    summary: Card 2's ImpactSummary names a lifecycle module that does not exist, instead of battencli and shedcli, the modules that build a shedverbs.Spec
---

The answer key is the decision record alone: no `_lyx/loom/rework/` directory exists, and the decision record carries no amendment.
Its Scope Out says "Tier 1 anomaly filing (`detectAndFileAnomalies`) stays `run`-only", and no Decision, Constraint or acceptance criterion mentions a retry of failed Tier 1 filings or the resumed-halt check.
The board entry `agent-self-reflect` does record the Tier 1 addition as made after Discussion, but the rubric binds the plan to the decision record, and the record was not changed to match.

### [BLOCKING] Tier 1 under step and the pending retry are unlicensed scope

**Location:** `_lyx/plan/07-anomalies-under-step.md`, `_lyx/plan/08-anomaly-pending-retry.md`, `_lyx/plan/00-overview.md` (summary line 11, Card Index rows 7 and 8, `### Decision: operator additions relayed by the hub after Discussion`, `### Decision: resumed-halt loss in Tier 1 — the finding`, `### Decision: who files a halt`, and the `loomCLI.fileIssue (card 7)` mention in `### Decision: one fakeable reflection seam`).
**Issue:** Card 7 runs `detectAndFileAnomalies` after every step, which the decision record's Scope Out excludes by name.
Card 8 adds a pending-anomaly retry to the filed-title marker, which no Decision or finding licenses.
The overview's own text concedes card 7 overrides the record ("overriding the record's Out item").
A plan that departs from its answer key cannot pass this gate; the key has to change first, upstream of Plan-Write.
**Fix:** Delete cards 7 and 8, drop their Card Index rows and renumber the later cards, remove the overview's summary sentence on Tier 1 and the three Tier 1 shared-decision sections, and reduce the fakeable-seam section to the reflection shuttle.

### [BLOCKING] Tier 1 scope threaded through cards 2, 6, 10 and 11

**Location:** `_lyx/plan/02-step-after-hook.md:22`; `_lyx/plan/06-halt-notes.md:32-34`, `:61`, `:63`; `_lyx/plan/10-reflection-stencil.md:26`; `_lyx/plan/11-friction-capture-docs.md:15`, `:24`.
**Issue:** Card 2's Intent cites "so loom can file Tier 1 anomalies" and "the hub-relayed Tier 1 addition".
Card 6's `writeHaltNote` takes `tierOneOn` and writes a Tier 1 ownership line, which D2's note content (producer, state, reason, history position, trace file path) does not include, with a test pinning it and a scenario sentence deferring to card 7's fake Tier 1 filer.
Card 10 adds a stencil rule that a halt note saying Tier 1 files the halt is not filed as a halt; D6 tells the reflection to file every distinct lyx problem, with no such exception.
Card 11 writes "Tier 1 anomaly filing runs under `step`" into the Friction Capture invariant, which D7 does not contain, and rewrites the `selfreport` module line to claim filing after every `step` and a retry.
Removing cards 7 and 8 alone would leave these, and card 11 would document behaviour no card builds.
**Fix:** Strip each: card 2's Intent cites only D2; card 6 drops `tierOneOn`, the ownership paragraph, its test bullet and the card-7 scenario sentence, keeping the statement that the `shedfake.Shuttle` stands in for the agent and so for its filing; card 10 drops the Tier 1 halt-note bullet; card 11 drops the Tier 1 clause from the invariant and the `selfreport` module-line bullet.

### [MEDIUM] Card 2's ImpactSummary names a nonexistent module

**Location:** `_lyx/plan/02-step-after-hook.md:54`.
**Issue:** The conclusion says "lifecycle leaves the hook nil and only gains an empty `friction` key".
No lifecycle module exists under `internal/`; the comments in `internal/shedverbs/spec.go` that name it are stale.
The modules that build a `shedverbs.Spec` are `internal/loomcli`, `internal/battencli` (its `armAt` wires `shedverbs.Hooks` without `AfterStep`, so its step envelopes gain an empty `friction` key) and `internal/shedcli`, which takes its `Spec` from `loomcli.ArmAt` or `battencli.ArmAt` through `shedcli`'s table and so inherits each recipe's envelopes.
**Fix:** Name `battencli` (hook nil, empty `friction` key on its step envelopes, opts out of error-path extras) and `shedcli` (delegates to the recipe's `ArmAt`, no code change) in place of lifecycle.

## Focus item 2: cross-references and the done-when filer

In the plan as read, every prose "card N" reference points at the right card, and card 6's done-when scenario states that the `shedfake.Shuttle` stands in for the agent and its `lyx selfreport create` filing.
That holds only for the current numbering: F1's fix removes cards 7 and 8, so cards 9 to 11 become 7 to 9, and the references to card 10 in cards 5 and 6 must follow.
This is not a finding against the plan as read.

## Rest of the plan

Cards 1, 3, 4, 5 and 9 carry D5, D1, D3 and D4 with stated bounds for each new seam.
The verify section runs and integration-tests every package a card targets, `internal/friction` and `contracts/stencils` included, and still does with cards 7 and 8 gone, since card 1 keeps `internal/loomengine` targeted.
The documentation constraint's list (`docs/overview.md`, `CONSTRAINTS.md`, the `internal/friction` and `internal/frictionengine` package docs, the loom recipe's Friction-Reflect comment, `InterruptPolicies`' doc) is carried by cards 11, 2, 1 and 3.
