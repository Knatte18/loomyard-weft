---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shedrecipe/entries_bouncer.go:372
    summary: No test checks that a harden_round Bouncer row sets FirstRaisedGates, so deleting the line that wires item 3 into the hardener fails no test
    fix: "In internal/shedrecipe/entries_harden_test.go, add a row to TestHardenRoundKey's harden_round Bouncer subtests that builds the Bouncer through bouncerEntry with harden_round true, runs two judge rounds as TestBouncer_JudgePromptGatesAFirstRaisedKeyWithFirstRaisedGates does, and asserts that round 2's decision rule lacks the clause \"on a key first raised in the latest round does not block it\"; add the counterpart row without harden_round that asserts the clause is present."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/bouncer/bouncer-template-judge.md:13
    summary: The judge stencil's maintainer comment still says the decision_rule marker renders the lighter rule from round 2 on, which a Bouncer with FirstRaisedGates set no longer does
    fix: "In the HTML comment at contracts/stencils/bouncer/bouncer-template-judge.md:13, after \"the lighter rule from round 2 on\", add \"unless the Bouncer sets FirstRaisedGates, which renders round 1's rule in every round\"."
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/burlercli/validate_integration_test.go:96
    summary: Test row names and a failure message still describe the removed focus-finding permission rather than the hardener-round exemption they now pin
    fix: "Rename the rows FocusFindingAdmittedInTheHardenerRunDirectory and FocusFindingRefusedInTheCodeRunDirectory in internal/burlercli/validate_integration_test.go:96-97 to OffChangesFindingAdmittedInTheHardenerRunDirectory and OffChangesFindingRefusedInTheCodeRunDirectory, rename the focusReview field to match, and in internal/shedrecipe/entries_burlerhalves_test.go's HardenRoundMarksTheRoundContext change the message \"want the focus finding admitted\" to \"want the off-changes finding admitted\"."
---

The diff carries out items 1 to 7 of the board entry and makes no code change for item 8, as the parent's added entry records.
The round check, the judge rule, the rubric and the review stencils agree with each other for a hardener round, and no non-hardener segment gets either flag.

### [MEDIUM] The harden_round Bouncer's FirstRaisedGates wiring has no test

**Location:** internal/shedrecipe/entries_bouncer.go:372

**Issue:** `bouncerCfg.FirstRaisedGates = true` is the only line that puts a hardener campaign on the stricter convergence rule.
The one test that names the field sets it by hand on a `BouncerConfig` (`internal/shedadapters/bouncerprompt_test.go`), and the `harden_round` Bouncer subtests in `TestHardenRoundKey` check the refusal, the seed and the facts, never the decision rule.
Deleting the line leaves every test green, so this behaviour the diff adds has no coverage.

### [LOW] The judge stencil's comment misdescribes the decision_rule variants

**Location:** contracts/stencils/bouncer/bouncer-template-judge.md:13

**Issue:** The maintainer comment says the marker "renders round 1's rule in round 1 and the lighter rule from round 2 on".
With `FirstRaisedGates` set, `decisionRuleMarker` renders round 1's rule in every round (`internal/shedadapters/bouncerprompt.go:28`).
The comment is stripped before the judge sees the stencil, so only a maintainer reads it, but it now states the old rule.
Line 44, "where the decision rule says so", still holds.

### [NIT] Test names still describe the focus-finding permission

**Location:** internal/burlercli/validate_integration_test.go:96

**Issue:** The rows `FocusFindingAdmittedInTheHardenerRunDirectory` and `FocusFindingRefusedInTheCodeRunDirectory` and the message "want the focus finding admitted" in `HardenRoundMarksTheRoundContext` name the `origin: focus` exemption the diff removes.
The rows still pass and still kill a missing flag, because their finding sits outside the changes file;
only the names are stale.

## Rejected

- Loosened checks on the removed `roundcheck_test.go` rows (lens:code-tests): `NewFocusDecisionBelowBlockingIsRefused` and `NewFocusFindingOfAnotherClassOnAnUntouchedFileIsRefused` pinned refusals that item 1 orders removed, so no assertion was widened to make a test pass. The behaviour itself changed. `NewFocusDecisionOnAnUntouchedFileOutsideAHardenerRoundIsRefused` pinned a refusal that `NewBlockingOnAnUntouchedFile` still covers.
- The BLOCKING-only rule lifted for hardener findings, flagged as unrequested scope (lens:code-scope): the study item 1 restates says c2's round 4 "could raise none, because an incremental round admits only BLOCKING findings on changed hunks" (`docs/research/hardener-works-study.md:156`). Item 3 also assumes a MEDIUM can be first raised in the closing round, which an incremental round with the BLOCKING rule cannot do. The lift is part of what item 1 asks for.
- Class-only gating, under which a MEDIUM `consistency` finding no longer blocks a hardener CONVERGED (handler): item 3 asks for one rule and leaves open which one. The change description's Design decisions names the choice, and the rubric's class rule sends every behaviour fault, a missed call site included, to `design`.
- Merge danger `two-way`, blast radius `hardener` (handler): the shared round check and judge rule change only under flags that only hardener rows set. The shared review stencils' new bullet is scoped to a hardener round. No record or board format, config meaning or outward push changes.
