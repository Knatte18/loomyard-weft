---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:34
    summary: D1 keeps only the order of two Go-filled markers, and D2 places marker-order rows that claims_test.go does not hold, so the rows pinning a Go-filled marker against prose or a heading fall between keep and drop
    fix: In D1's first keep bullet, replace "the order of two markers Go fills" with "the position of a Go-filled marker relative to another marker, a heading, or the end of the body"; in D2 replace "marker-order rows join `markers_test.go`" with "rows pinning a Go-filled marker's position (the `{{.kickstart_pack}}` end-of-body row and the `{{.kickstart_pack_inline}}`-before-`## The sandbox rule` row) join `markers_test.go` as one table of marker positions", or, if the operator line is that such placement is prose, name both rows under D2's drop bullet instead.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:46
    summary: Deleting TestStencilClaims drops its //lyx:guard, and no Decision says whether D2's Go-value table and the host-neutral scan take the marker, so what the card gates stop catching is unstated and unbounded
    fix: Add a bullet to D2 stating that the Go-value table test and the twill host-neutral scan each carry `//lyx:guard`, so a card that renames `shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName` or `gatecli.SlotBusyExit` outside `contracts/stencils` still fails its own card gate; then delete the second bullet of Notes for the plan writer, which D2 now covers. If the operator prefers no guard marker, state instead in D2's rationale that such a rename passes every card gate and is caught first by the plan-level verify.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: D4's bound names Plan-Review, which reads the plan and gains no D4 bullet, and omits landings that pass no review rubric at all, such as the orchestrator's own pushed fix on main
    fix: Rewrite D4's rationale sentence to "This downgrades a guard from mechanical to judged. What it lets through is a prose regression the reviewer misses, and any stencil change landed outside loom and darn, such as the orchestrator's own pushed fix or an operator commit on main, which no rubric reads. What bounds it is that a loom landing passes Webster-Review and a darn landing passes Darn-Review, each reading stencil changes against the stick it already uses."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:109
    summary: Acceptance criterion 2 says rewording any static prose sentence fails no test, which D1's own keeps contradict, since branch anchors and the PATTERN scans over stencils pin prose phrases
    fix: Replace the criterion with "Rewording a sentence of static prose in any stencil fails no test, unless the sentence holds a marker, a Go-named value, a twill host word, the phrase a kept test anchors a Go-chosen branch on, or text a PATTERN scan over stencils checks."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:66
    summary: D3 exempts stencil prose from the no-coverage item in the webster-review and darn-review rubrics but not in loom-rubric-merge-review.md, whose Test economy list still asks for coverage of every interaction behavior
    fix: Extend D3's fifth bullet to name `loom-rubric-merge-review.md` alongside the webster-review and darn-review rubrics, so its "a behavior of the interaction with no coverage" item exempts stencil prose too, and add that rubric to acceptance criterion 8's list.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:120
    summary: The note lists class and first_card as literal key rows without a disposition, though their only Go definitions are unexported structs a contracts/stencils test cannot reference, so D1 drops them
    fix: Rewrite the note's second clause to "its literal key rows, such as `class` and `first_card`, go by D1, since their Go definitions (`loomshed`'s `roundRecord`, `planparser`'s `overviewFrontmatter`) are unexported."
---

### [MEDIUM] Marker placement against prose falls between D1 and D2

**Location:** `_lyx/discussion/decision-record.md:34`, `_lyx/discussion/decision-record.md:51`

**Issue:** D1 keeps "the order of two markers Go fills", and D2 sends "marker-order rows" to `markers_test.go`.
`claims_test.go` holds no row ordering two markers: the claim type has only `must` and `mustNot` phrases.
What it does hold is two rows pinning one Go-filled marker against prose (`contracts/stencils/claims_test.go:580` and `:582`): `{{.kickstart_pack}}` ends the body, and `{{.kickstart_pack_inline}}` precedes `## The sandbox rule`.
They are not section-scoped, so D2's drop bullet does not name them, and D1's order keep does not match them.
The same shape sits outside `claims_test.go` in `internal/burlerengine/template_test.go:211`, which pins a Go-filled directive before the `## What to review (the target)` heading, so the gap reaches D1's general sort as well as D2's placement.

### [MEDIUM] The dropped `//lyx:guard` has no stated bound

**Location:** `_lyx/discussion/decision-record.md:46`

**Issue:** `TestStencilClaims` carries `//lyx:guard`, and `internal/websterengine/cardgate.go:88` runs every guard test in every card gate, whatever packages the card targets.
That is what today fails a card in `internal/shuttleengine` that renames `MessageTail` without the stencil.
D2's rationale says such a rename "breaks a run … unseen", yet no Decision says whether the new Go-value table or the host-neutral scan carries the guard marker.
The only mention is the second note for the plan writer, which says the drop happens "with no other change": a removed guard with no stated bound, the attack-surface question's case.

### [MEDIUM] D4's bound overstates its coverage

**Location:** `_lyx/discussion/decision-record.md:79`

**Issue:** Plan-Review reads the plan against the decision record, never the diff, and D4 adds its bullet only to the webster-review and darn-review rubrics, so Plan-Review guards no stencil wording.
The bound also says nothing about landings outside loom and darn, which `CLAUDE.md` permits (the orchestrator's own pushed fix on a clean `main`).
Before this change, `TestStencilClaims` caught a dropped protected claim on those routes too, through the push-to-main gate run, so they are exactly what the downgrade now lets through.

### [MEDIUM] Acceptance criterion 2 contradicts D1's keeps

**Location:** `_lyx/discussion/decision-record.md:109`

**Issue:** D1 keeps an assertion that tells Go-chosen branches apart, "anchored on the shortest phrase that tells the branches apart", and that phrase is stencil prose.
The PATTERN scans D1 keeps match prose too: `TestStencils_NoUnboundedAskOperator` and `TestStencils_NoEditRuleRestatement` in `contracts/stencils/parentdirective_test.go` fail a reword that asks the operator or restates the edit rule, which the third note for the plan writer confirms.
As written, the criterion cannot be met, and a plan writer who treats it literally would drop kept assertions to satisfy it.

### [LOW] The merge-review rubric keeps an unexempted coverage item

**Location:** `_lyx/discussion/decision-record.md:66`

**Issue:** `contracts/stencils/loom/loom-rubric-merge-review.md:59` lists "a behavior of the interaction with no coverage" under **Test economy**.
D3 exempts stencil prose from the matching item in the webster-review and darn-review rubrics only, so a Merge-Review round whose merged interaction is a stencil change can still demand a prose test.
The focus question asks whether any reviewer still demands the tests D1 drops, and this rubric does.

### [LOW] The `class` and `first_card` note leaves their fate open

**Location:** `_lyx/discussion/decision-record.md:120`

**Issue:** The rows at `contracts/stencils/claims_test.go:154`, `:155`, `:181` and `:182` pin `class` and `first_card` as literals.
Their Go definitions are the unexported `roundRecord` (`internal/loomshed/prrework.go:80`) and `overviewFrontmatter` (`internal/planparser/parse.go:86`), so no `contracts/stencils` test can read the tags, and D1's drop clause applies.
The note names them right after the Go-value rows D2 keeps, which reads as a keep.

## Rejected

- Focus 1, the D1 keep categories other than marker placement: the Go-filled-value and branch keep, the Go-named constant keep and the cross-stencil keep each map to a D2 bullet or stay in their own template, prompt or scan tests; F1 is the only row that falls between.
- Focus 2, D5: lowering no budget row removes no guard and opens no edge, so the attack-surface question does not apply.
- Focus 5, deliberation narrative: D1's rationale cites the operator decision and the board's open item, and D4's names the board option; each is one clause of provenance the plan writer can use, and Open risks and Notes carry actions, not deliberation. No relocation finding.
