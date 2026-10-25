---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: docs/overview.md:569
    summary: "non-Go files naming removed names are still found one by one, round after round; docs/overview.md keeps the retired parent_review_wait_min key and Discussion-Write's parent-review gate in the status waiting note, and no card edits either line"
    fix: "Make each removing card list its non-Go references by sweep, not by memory: for the names a card removes, search the non-Go files outside docs/research and docs/benchmarks and give every hit a card. For the two hits found now, in _lyx/plan/06-retire-alignment-keys.md add a **Prosa:** group holding `docs/overview.md` (placed before **Delete:**) and one Intent bullet saying the loom config paragraph (docs/overview.md line 569) drops `parent_review_wait_min` from the keys it lists and the reconcile paragraph after it (line 570) names the four retired keys among those dropped. In _lyx/plan/04-drop-review-verbs.md extend the existing Docs bullet, which already targets `docs/overview.md`, so that the `status` waiting-note sentence (line 515, 'Discussion-Write's parent-review gate') stops naming the gate. Both edits pass `lyx loom validate-plan --overlay` with the same result as the baseline."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/16-parent-align-row.md
    summary: "card 16 enumerates the fixtures that hold the gate key merge-in by hand and still leaves two, TestLoadConfig's scope-refusal rows and the gates block of roundGateConfig, so the old key survives in test data"
    fix: "Derive the fixture list by a sweep of the test files for the row name and gate key literals, and give every hit a glyph. For the two hits found now, in _lyx/plan/16-parent-align-row.md add `internal/gateslot#TestLoadConfig` and `internal/loomshed#roundGateConfig` to **Edit:** after `internal/gateslot#TestWriteRunRecord`, and extend the fixture sentence at the end of the Intent so it also names TestLoadConfig's two scope-refusal rows keyed merge-in (config_test.go lines 125 and 126, including the quoted gates.merge-in key in the want text) and the gates block of roundGateConfig (loomshed/verifygate_integration_test.go line 146). Both glyphs resolve with `lyx quarry resolve`, and the combined overlay passes `lyx loom validate-plan --overlay` with the same result as the baseline."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/12-bouncer-decision-escalation.md
    summary: card 12 adds a routing edge whose accept lets BLOCKING findings through, yet its Intent never states what it lets through or what bounds it, and card 8 does not say a mis-classed decision stops the run
    fix: "In _lyx/plan/12-bouncer-decision-escalation.md add, before **ImpactSummary:**, a paragraph opening 'What it can skip or let through:' (an accept passes the round only over a judgment the refusal admits, carrying the listed findings into ## Open risks as a circling accept does; a continue starts the next round only over a ledger that keeps every key of the previous round's) followed by one opening 'What bounds it:' (the class definition card 8 writes is reviewer discipline only, so a decision mis-classed by a reviewer stops the run until the parent answers; one escalation per judged round plus one per retired decision at the budget below the hard cap; the hard cap, at which a held refusal halts the run; the bounce budget the re-judge spends). The text change passes `lyx loom validate-plan --overlay`."
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/07-scope-lenses.md
    summary: card 7 plans new assertions on the static prose of two lens texts, which the plan-writer stencil says no test covers
    fix: "In _lyx/plan/07-scope-lenses.md reduce the first Tests bullet to the structural assertion that `review-discussion` holds the six entries in the stated order, dropping the added sentence checks for goal-scope (Decision for <slug>: and class decision) and board-fit (lyx loom peers and the 'spends one of its four reads on a note' sentence); keep the TestLoad case, which pins behaviour (a tuned lens keeps its own text). The text change passes `lyx loom validate-plan --overlay`."
---

### [MEDIUM] docs/overview.md keeps two statements the plan makes false

**Location:** `docs/overview.md:569` and `docs/overview.md:515`; cards 6 and 4.

**Issue:** Card 6 retires `parent_review_wait_min` from `loom.yaml`, but line 569 of `docs/overview.md`, the loom config module paragraph, lists it among the keys the module holds, and line 570's reconcile paragraph names the dropped keys without the four new ones.
Card 4 removes the status waiting note for Discussion-Write's parent-review gate, and card 3 removes the gate, but line 515 still says `status` carries a waiting note while a run waits on "Discussion-Write's parent-review gate".
Card 4's Docs bullet names only the verb list and the package list of that file, card 5 only the module tree, card 17 only the Merge-In text, and card 6 does not target the file at all.
The decision record's D2 requires that no text name parent review for the step, and the repository rule that docs land in the same commit applies to an observable behavior change.
A search of the non-Go files for the removed names finds both lines.
Round 1's F8 was the same kind of gap, a `pattern/` file the plan named nowhere, and its fix added that one file; the method of listing non-Go references from memory is what keeps missing, so this is filed as `design`.

### [LOW] Two gate-key fixtures keep merge-in

**Location:** `internal/gateslot/config_test.go:125`, `internal/gateslot/config_test.go:126`, `internal/loomshed/verifygate_integration_test.go:146`; card 16.

**Issue:** Card 16 ends its Intent with an explicit list of the test fixtures that hold the row name `Merge-In` or the gate key `merge-in` as a string literal.
The list omits `TestLoadConfig`, whose two scope-refusal rows write `gates:\n  merge-in: ...` and expect the text `gate config key "gates.merge-in"`, and the `roundGateConfig` constant, whose `gates` block still keys `merge-in`.
Neither breaks, because the loader accepts any gate key, so the omission would pass the gates and leave the old key in the data the new key is meant to replace.
Round 1's F3 was the same gap, and its fix added the fixtures found then to the list; a hand-built list keeps missing, so this is filed as `design`.
Every code reference to the deleted identifiers (`NameMergeIn`, `landingshed.MergeIn`, `NewMergeIn`, `FixerModeMergeIn`, `mergeInEntry`, `mergeInName`) sits in a function the card names, which I checked function by function.

### [LOW] Card 12 states no lets-through and bound for the decision escalation

**Location:** `_lyx/plan/12-bouncer-decision-escalation.md`

**Issue:** The card adds a routing edge that can stop a run for a parent answer, and an accept that passes a round over BLOCKING findings, which is a guard the judge's CONVERGED verdict used to hold.
The Intent describes the mechanics and the switch that turns them off, but never states in one place what the edge lets through and what bounds it, as the decision record's D8 closing paragraphs do and as cards 3, 7, 10, 13 and 14 do for their own changes.
Missing in particular is the bound that the class definition is reviewer discipline only, so a writer's choice mis-classed `decision` stops the run until the parent answers; card 8 says only that the class goes to the parent.
The rubric treats an introduced edge with no stated bound as a finding.

### [LOW] Card 7 pins lens prose with new test assertions

**Location:** `_lyx/plan/07-scope-lenses.md`

**Issue:** The first Tests bullet extends `TestConfigTemplate_DecodesAndIsSelfConsistent`'s sentence check with exact phrases of the two rewritten lens texts, such as `spends one of its four reads on a note`.
Lens text is read by agents at call time like a stencil, and the plan-writer stencil states that a stencil's static prose is not behavior a test covers.
The existing sentence check for older lenses is a precedent, not a requirement of the decision record, and the rewrite's behavior is already pinned by the six-entry fan order and the `TestLoad` case.

## Rejected

- Decision halt routing (focus 1): card 14 withholds the prefixed halt from every consumer I traced.
  `arbiterstop.Check` sets `DecisionHalt` and `Eligibility` refuses it with `HandBackWayForward`; `HoldReading` takes its eligibility from `Check`, so batten's hold follows.
  `battenCLI.wire`'s `HandBack` seam, `loomCLI.specFor`'s `Spec.HandBack` (which marks the loop envelope's `hand_back`) and `goDriverDepsFor`'s `Deps.HandsBack` all move to the new predicate.
  `decideDrive` tests `stop.HandBack` ahead of the budget, precondition, arbiter and tackler rows, so the sheddriver tackler path never sees the halt.
  The Reason survives `stuckReason` unchanged, so a prefix match on the persisted error holds, and `arbiterstop` already imports `shedadapters`, so the card adds no import cycle.
- Rename to Parent-Align (focus 2): no Go identifier of the old row is left uncovered.
  The two literal gaps are F2; the other `merge-in` literals I found are free-form fixtures (`FixerAttemptsPath` row-name arguments, a `LandingFailure` record's `row`, a reroute-cap sentence) or fabric's own verb, which D11 keeps.
  `merge_fix_attempts` appears only in files cards 15 and 17 name.
- Decision-cause escalation (focus 3): card 12's routing, carry-over and held-refusal brief match D8.
  The accept leaves `decision` findings out of both BLOCKING checks and an open BLOCKING entry with no class still refuses; a continue applies the dropped-key check only; a refusal below the budget re-judges, one at the budget below the hard cap retires the decision and re-escalates, and one at or past the hard cap halts with a budget-exempt `Halt` Stuck whose Reason names `lyx loom resume`.
  A settled `decision` accept stands on a replay and re-settles or starts a new generation at entry (tests 13 and 18), and the BLOCKING `decision` finding alone does not refuse the accept (test 9).
  The carry-over adds the BLOCKING findings the ledger closed and the `decision` findings at any severity, matching by exact key.
  The only gap is the missing statement in F3.
- Non-Go files naming deleted or renamed names (focus 4): the stencils, `loom-recipe.yaml`, `shed-recipe-spec.md`, `loom-status-spec.md`, `twill-directive.md`, the two orch stencils, `pattern/PATTERN-cwd-resolution.md`, `pattern/PATTERN-shuttle-provider-seam.md`, `internal/loomengine/template.yaml` and the module tree of `docs/overview.md` are each edited or deleted by a card.
  The remaining gap is the two lines of F1.
- Remaining text-only leftovers: some test-file comments still say `Merge-In`; D11 scopes the doc renames to recipe comments, stencils, specs, `pattern/` files, README, `docs/overview.md` and package docs, so they are no finding.
- Mechanical checks, card order, member glyphs and re-sign arrows, `Extends` use, `Custom` use, verify lines and PATTERN fit raised nothing beyond the above.
