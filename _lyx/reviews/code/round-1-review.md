---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shedadapters/burlerfix_test.go:533
    summary: the rule that the notice is never posted on the resume of a live fixer has no test
    fix: in internal/shedadapters/burlerfix_test.go add a row to TestBurlerFixProducer_Call_ProbeAndResume, or a live-fixer row to TestBurlerFixProducer_Call_SendsDecisionFindingsToTheParent, with a live fixer, a told notifier and a review holding a decision-class finding; assert the notifier received no line and the fixer was resumed, not started.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/shedrecipe/entries_burler.go:197
    summary: the hand-off of Env.NotifyParent into BurlerHalfDeps in burlerFixEntry has no test, so dropping it silently disables the routing
    fix: extend the existing shedrecipe test that builds BurlerFix rows so a row built with a told Env.NotifyParent, once called over a review holding a decision-class finding, posts through it; leave the wiring closure in internal/loomcli/wiring.go uncovered unless an existing wiring test reaches it cheaply.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: contracts/stencils/orch/orch-template-role.md:73
    summary: the orch role stencil gains a bullet for the findings notice, a change the decision record's In scope does not list
    fix: either remove the added bullet from contracts/stencils/orch/orch-template-role.md, or leave it and have the parent record it as an addition to the scope with lyx loom decision add; the notice text already names the way forward, so the bullet is not needed for the acceptance criteria.
    origin: handler
  - id: F4
    severity: LOW
    class: consistency
    location: contracts/stencils/webster/webster-body-implementer.md:11
    summary: the implementer and verify-fix bodies repeat the notice opening as a literal that no test ties to decisionNoticeOpening
    fix: in contracts/stencils/webster/webster-body-implementer.md and webster-body-verify-fix.md:25 replace the quoted phrase with a pointer to the decision notice the parent directive admits, so only the parent directive stencil, which TestDecisionNoticeOpening_NamedByParentDirective pins, holds the phrase.
    origin: lens:code-affected-set
---

### [MEDIUM] Resume of a live fixer is not pinned against a notice

**Location:** internal/shedadapters/burlerfix_test.go:533
**Issue:** D1 and the acceptance criteria say the notice is posted once per fresh fix call and never on the resume of a live fixer. `Call` honours that by sending after `probeLiveFix` returns, but no test has a live fixer with a notifier and a decision-class review, so moving the post above the probe breaks nothing.

### [LOW] NotifyParent hand-off in the BurlerFix entry is untested

**Location:** internal/shedrecipe/entries_burler.go:197
**Issue:** the only test that mentions `NotifyParent` builds the producer directly. A change that stops `burlerFixEntry` passing `env.NotifyParent` on leaves every test green and turns the whole routing off in loom and darn runs.

### [LOW] Orch stencil change outside the recorded scope

**Location:** contracts/stencils/orch/orch-template-role.md:73
**Issue:** the record's In list names the parent directive, the Master stencil and the implementer and verify-fix bodies; the orch stencil is not among them, yet the diff adds a bullet there and the change description lists it. The bullet is consistent with D1's way forward, so this is a scope departure, not a defect.

### [LOW] Notice opening duplicated in two stencils with no pin

**Location:** contracts/stencils/webster/webster-body-implementer.md:11
**Issue:** both bodies quote `Parent decision recorded with lyx loom decision add` literally. The loomcli test pins only the parent directive's copy to `decisionNoticeOpening`, so a change to the constant leaves these two copies stale with no signal.

## Rejected

- Multi-line decision text breaking the creel post (code-correctness, unverified): creel turns line breaks and control characters into one space, per `internal/creelengine/doc.go`.
- `TestDecisionNoticeOpening_NamedByParentDirective` as a new assertion on stencil prose (code-tests, NIT): it is the one pin between the constant and the directive, and the record's acceptance criteria ask for the rendered directive to name the phrase.
- `Env.NotifyParent` as a row `Env` field against `PATTERN-recipe-setting-lookup` (code-pattern-fit): it is a run-wide callback like `ParentName`, not a per-row value, and the record's constraint asks for it to reach the row through `shedrecipe.Env`.
- The skipped-fix Info log saying the review held no findings on a routed skip (code-correctness): a log line only.
