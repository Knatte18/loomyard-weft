---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:63
    summary: D5's Bound names only what an entry cannot loosen, never what it can skip, though a compile-only entry removes a tier from the plan-level verify gate's run
    fix: In D5's Bound, add that an entry calling a tier compiled rather than run lets the plan's `## verify:` section, which webster's plan-level verify gate runs as written, skip running that tier, and lets Plan-Review accept the skip; state that this is bounded by the key's Guard classification on a board-only file and by the Go-coded card and batch gates, which still run the untagged and `-tags integration` tests over the batch's packages whatever an entry says.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:33
    summary: D1's rubric bullet limits a finding's licensed scope to the decision record, while D1's rework stencil plans every non-overturning finding with no recording, so a finding that adds scope outside the record is planned by one and flagged by the other
    fix: In D1, decide one rule for a finding that adds scope the record does not cover without overturning a Decision, and state it in both the rework stencil sentence and the rubric bullet — either such a finding licenses its card's scope as today (rubric text says a finding licenses scope unless it overturns a Decision), or it takes the same parent-recording route as an overturning finding (rework sentence and Bound say so).
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:31
    summary: D1's stop-and-report on a declined or unrecordable finding leaves open whether the whole rework stops or only that finding, against the rework stencil's rule that every finding is covered by a new card and coverage.md maps each one
    fix: In D1, state whether the stop ends the rework session without writing coverage.md (the other findings unplanned, the run halting for the parent) or only drops that finding, and in the latter case how coverage.md records it and why Plan-Review's rework answer key does not flag it as uncovered; mirror the choice in the Acceptance criteria bullet at line 130.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: The implementer's empty-list fallback holds back no tier from a raw run, which for this Go hub tells the implementer to raw-run the tmux and llm tiers the kept shell-guard row refuses, so D7's claim that the neutral rule stays correct for a Go hub is false and Open risks omits it
    fix: In D6's implementer bullet, make the empty-list fallback name no tier as run-allowed beyond what the kept guard sentences allow (for example, a raw run of the listed packages only, any run the shell guard refuses going through `lyx gate test`), or state the stencil keeps the guard-describing sentence for the tmux and llm refusal; then correct D7's rationale and add the implementer case to Open risks if any gap remains until the operator sets the value.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:79
    summary: D6's implementer rule says a tier the list calls compiled rather than run is never run, but does not say the rendered card and batch gate steps still run as written, so an entry naming a tier the batch gate runs gives the implementer two contradicting instructions
    fix: In D6's implementer bullet, state that the existing sentence that a `lyx gate test` step runs as written, whatever its `--tags`, stays, and that the list's compile-only rule governs raw runs and runs no gate step names only.
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:72
    summary: D6 names the empty-list render only for the rubric (`none`); for the plan prompt and both implementer value maps it does not say whether the marker is optional and renders nothing or renders `none`
    fix: In D6, state for the plan template and the implementer body how the empty list renders — an optional marker in the fill's optional list rendering nothing, or a `none` value — so the fallback sentence reads correctly beside it; extend the empty-list Acceptance bullet at line 128 to say each render reads correctly, not only the rubric.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:130
    summary: Acceptance pins D1's rework prompt and its decline outcome but neither D1's Plan-Review rubric bullet change nor the no-parent-recorded case
    fix: Extend the Acceptance bullet at line 130 so the rendered Plan-Review rubric calls a finding a reviewer's claim and flags a card carrying a Decision-overturning finding with no `Added after Discussion` entry, and so the rework prompt stops and reports when no parent is recorded as well as on a decline.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:61
    summary: D5's rationale restates the rejected structured schema and carries the .NET mapping deliberation, which belong to the support log and carry no requirement the plan writer needs
    fix: Move the sentence at line 61 (prose over a per-tier schema) and the .NET mapping sentence at line 62 into the support log's Turn 5 or Rejected alternatives entries, which already hold the schema rejection and the dotnet-verify-protocol check; keep in D5's Rationale only that the tiers are facts of the hub's language and `language.yaml` is the hub-wide module holding language-specific skills.
---

### [MEDIUM] D5's Bound does not state what an entry can skip

**Location:** `_lyx/discussion/decision-record.md:63`

**Issue:** The rubric's attack-surface question asks each new routing edge to state what it can now skip or let through and what bounds it.
D5's Bound says entries reach the prompts verbatim and cannot loosen the shell guard or the plan gate.
But D6 makes an entry that calls a tier "compiled rather than run" exempt from Plan-Review's coverage finding and tells the plan writer to follow the list's form.
Webster's plan-level verify gate runs `plan.Verify` as written (`internal/websterengine/verifygate.go`), so such an entry removes that tier from the gate's run.
`configengine.CategoryGuard` is defined as a key that "hollows a guard, a review or an approval gate", which is what this does, and the plan gate's `verify-module-wide` and `verify-nested-module` checks do not touch compile-only tiers.
What bounds it, the Guard classification and the Go-coded card and batch gates (`internal/websterengine/doc.go`: card gates over card packages, the batch gate with `-tags integration`), goes unstated.

### [MEDIUM] D1's rubric scope rule disagrees with its rework rule

**Location:** `_lyx/discussion/decision-record.md:33`

**Issue:** D1 says every finding but a Decision-overturning one "is covered by a card with no recording, as today".
The rubric bullet it prescribes says a finding "licenses a card's scope only within the decision record".
The current rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`, Fidelity bullet) licenses a card's scope "by the decision record or by a finding".
A finding that adds work outside the record without overturning a Decision is therefore planned freely by the rework writer and flagged by Plan-Review.
The plan stencil's scope-addition route covers additions after Discussion, so it is unclear which rule D1 intends.

### [MEDIUM] D1's stop-and-report outcome is unscoped

**Location:** `_lyx/discussion/decision-record.md:31`

**Issue:** The rework stencil requires every finding covered by a new card and coverage.md mapping each finding, and coverage.md's existence is its done signal.
D1's "the writer stops and reports the finding" does not say whether the session stops whole or only that finding is set aside.
The Bound's "keeps a declined finding from stalling the round silently" reads as a session stop, but a decline means the parent judged the finding should not override the record, which reads as dropping only that finding.
The two readings produce different plans and different coverage.md content.

### [MEDIUM] The implementer's neutral fallback conflicts with the kept guard for a Go hub

**Location:** `_lyx/discussion/decision-record.md:79`

**Issue:** D6's implementer fallback: "With no list, no tier is held back from a raw run on the listed packages."
The Out list keeps the shell guard's raw-go row, which refuses raw `tmux`-tier and `llm`-tier runs, and the current implementer text compiles those tiers through `lyx gate test ... -run '^$'` (`contracts/stencils/webster/webster-body-implementer.md`, raw-run section).
Until the operator sets this hub's value, implementers here are told raw `llm` and `tmux` runs are allowed, then refused by the guard.
D7's rationale says the neutral rule "stays correct for a Go hub", which does not hold for the implementer, and Open risks names only the plan writer and Plan-Review.

### [LOW] Compile-only rule versus rendered gate steps

**Location:** `_lyx/discussion/decision-record.md:79`

**Issue:** The current section says "A `lyx gate test` step runs as written, whatever its `--tags`", which is what keeps a compile-only rule from overriding a gate step.
D6 lists what stays in the section ("The slot-busy exit text stays") and does not name this sentence.
Without it, an entry calling a tier compile-only contradicts a batch gate step that runs it.

### [LOW] Empty-list render unstated for the plan and implementer

**Location:** `_lyx/discussion/decision-record.md:72`

**Issue:** The plan template is filled by `internal/seatengine` through `stencil.FillWith` with an optional list, and the implementer body through `stencil.FillOptional` (`internal/websterengine/render.go`), and both refuse an empty required marker.
D6 settles the rubric's empty value as `none` but leaves the plan and implementer marker's empty form open, and the Acceptance bullet checks that only the rubric reads correctly.

### [LOW] Acceptance does not pin D1's rubric change or the no-parent case

**Location:** `_lyx/discussion/decision-record.md:130`

**Issue:** D1 changes the Plan-Review rubric's rework answer-key bullet and has the writer stop when no parent is recorded.
No Acceptance bullet covers the rubric bullet, and the rework bullet names only "stops on a decline".

### [NIT] D5's rationale carries deliberation

**Location:** `_lyx/discussion/decision-record.md:61`

**Issue:** "Entries are prose an agent reads, so a structured schema per tier would encode Go's shape" restates the support log's first Rejected alternative.
The .NET sentence records the lens review's dotnet-verify-protocol check; its only requirement, that `verify_tiers` is prompt guidance only, is already in Scope Out.
Neither carries a requirement or constraint the plan writer needs, so the move loses nothing.

## Checked and not flagged

- False premises: `languageconfig.Load` uses `configengine.LoadOrTemplate`, classifies `skills` `CategoryGuard`, and refuses a non-list, a non-string, an empty entry and surrounding whitespace with the `lyx config language` way forward; `configengine.LocalKeys` derives from `CategoryNone`, so a Guard key stays out of it; `shedadapters.ReadRubric` fills with strict `stencil.Fill`, which refuses an empty marker; `hubreconcile.Ensure` reconciles the hub-wide config; `applyReject` in `internal/landingshed/pralign.go` records the alignment agent's report as findings; `languageconfig.Load`'s callers are the burler, loom and webster CLI wiring and `configreg`'s `CheckLoad`.
- D2, D3 and D4 match the cited stencil text in `shed-template-driver.md`, `loom-template-discussion-advisor.md`, `loom-template-plan.md` and `loom-template-rework.md`.
- Every Acceptance bullet traces to a Decision.
- Auto-mode assumptions and Open risks hold no deliberation that belongs in the support log.
- No Decision edits this task's own board entry.
