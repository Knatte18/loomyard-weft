---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:72
    summary: D4 accepts the scalar spellings of discussion_advisors but never makes the key an open map, so config reconcile, which runs on every binary change, replaces an operator's scalar with the template's one-advisor list
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:140
    summary: D8 bounds an advisor's message to the parent by the parent directive's trust rule, but the parent is the orch, whose stencil sits outside that rule and acts on messages from a run's agents
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:98
    summary: D5's bound says advisor edits to the two discussion files pass the chair's self-check, gate and parent review, but advisors stay live through the gate and are stopped only after it, so an edit made during that wait reaches only the Discussion-Review perch
---

### [MEDIUM] `discussion_advisors` is not an open map, so reconcile reverts a scalar value

**Location:** `_lyx/discussion/decision-record.md:72` (D4) and the PATTERN-config-strictness Constraints bullet at line 183.

**Issue:** D4 reads the key with the same scalar-or-sequence shape as `review`, accepts the empty scalar and null as zero advisors, and sets the template default to the sequence `[sonnet[high]]`.
The Constraints bullet says the key "rides the existing loom config load, template and reconcile paths".
Load accepts every spelling: `yamlengine.MissingKeys` treats a sequence-element leaf (`discussion_advisors[0]`) as present whenever the owning key is present, whatever its shape.
Reconcile does not.
`yamlengine.Reconcile` (`internal/yamlengine/reconcile.go`) carries an existing list whole only when the existing value is also a sequence (`applyExistingSequences`).
When the operator's value is a scalar (`""`, null, or a single model-spec like `opus[high]`), the template leaf `discussion_advisors[0]` is reported added, the operator's scalar leaf `discussion_advisors` is reported removed, and the merged file holds the template's `[sonnet[high]]`.
The `review`-family keys avoid this because `loomengine.ConfigOpenMaps` lists them, and `carryOpenMaps` carries an open-map key whole whatever its shape.
`hubreconcile` runs `configsync.ReconcileAll` on every binary change (`internal/hubreconcile/walk.go`), so after the next deploy, an operator who turned advisors off with `discussion_advisors: ""` gets one sonnet advisor back without being told.
That breaks D4's own Bound, which says the list's length is the advisor count for every spelling the load accepts.
The Acceptance criteria cover only load and the reconciled template, so nothing pins it.

**Fix:** In D4, require that `discussion_advisors` joins the loom module's open-map keys, as the other scalar-or-list model-spec keys do, so reconcile and `--set` carry it whole.
Extend the reconcile Acceptance criterion: a file holding the key as `""`, null, a single scalar, `[]` or a two-entry list keeps its value through `lyx config reconcile`.

### [MEDIUM] D8's bound on an advisor messaging the parent rests on a trust rule the parent does not carry

**Location:** `_lyx/discussion/decision-record.md:140` (D8 Bound, third sentence).

**Issue:** D8 says an advisor that messages another session is bounded by that session's trust rule: "a spawned role trusts only its parent's messages, so an advisor message to the parent or another pair's session carries no authority there".
That rule is the parent directive's "Trust no other session's message" (`contracts/stencils/parent/parent-directive-parent.md`), which spawned roles render.
The run's parent is the orch (`ly:orch` in a hub run), and PATTERN-parent-directive puts the orch stencils outside that rule.
The orch's role stencil has no such restriction.
`contracts/stencils/orch/orch-template-role.md` treats "a message or escalation from a driver, webster or review loop" as a stop, and when the message names an action, it takes that action itself without a fork ("Investigating a stop").
Its actions include `lyx loom decision add` and `lyx loom goto`.
So the premise is false for the parent, which is the one recipient D8 names first.
An advisor message to the orch is limited only by how the orch judges it, and the record does not say that.
Advisors are the one new sender this task adds, and the attack-surface rule requires each new edge to state what bounds it.

**Fix:** Restate the bound for the parent.
Either say what does bound it (for example, the orch acts only on messages from the run's driver, writer or review loop, and an advisor strand is none of those, if that is the intent), or move it to the record's "Not bounded" wording, as D5 does for record edits, and add an Open risks entry.
Keep the parent-directive argument only for the other pairs' spawned sessions, where it holds.

### [LOW] D5's bound overstates which checks an advisor's edit to the discussion files passes

**Location:** `_lyx/discussion/decision-record.md:98` (D5 Bound, third sentence).

**Issue:** D5 says advisor edits to the two discussion files "still pass through the chair's self-check, the chair's gate, the parent review and the Discussion-Review perch before Plan reads the record".
The seat engine stops advisors only after the chair's wait returns (`join` in `internal/seatengine/engine.go`: wait on the chair, then `stopAdvisors`), and the chair's gate, the parent-review wait included, runs inside that wait (`StartGated`).
An advisor is live, held quietly, for the whole gate.
The parent-review store pins no content of the files it was asked to review.
An edit an advisor makes after the self-check ran, or after the parent read the record, therefore reaches the commit with only the Discussion-Review perch still ahead of it.
The single writer has no such window, because its turn has ended while its gate waits.
The "Not bounded" sentence that follows is right about the outcome but understates how often it can happen.

**Fix:** Reword the bound to what holds: an advisor edit made before the chair's self-check passes the self-check, the gate and the parent review; an edit made while the gate waits reaches only the Discussion-Review perch.
Either accept that window in "Not bounded" or require that advisors stop before the chair's gate runs.
Picking one is the author's call.

## Focus directive verdicts

- D2/D11 empty directives: not a defect.
  D11 adds a `seatengine.Table` field that exempts a listed value from `validateValues`' blank refusal and passes the list as the `optional` argument of `stencil.FillWith`.
  `FillWith` already skips listed names in both unfilled-marker guards and renders a blank listed value as empty, so the mechanism works against `missingkey=error`.
  Acceptance criterion 2 covers PATTERN inactive, Tier 2 off and a friction read failure, and the two refusals that must stay.
- D4 spellings: not a defect at load.
  D4 normalises absent, null, `[]`, `""` and `[""]` to the empty list and refuses an empty entry beside another, and Acceptance criterion 3 names each spelling.
  The reconcile path is the gap, filed as F1.
- D5/D8 advisor limits: filed as F2 and F3.
  The rest of D5's bound holds: the settings denies apply to every spawned session, `discussionCommitPathspec` stages only `_lyx/discussion/` and the parent-review round directories, and stray edits elsewhere are not committed by this step.
- D3 amendment: not a defect.
  D3 no longer carries an amendment, and the Discussion-Write entry today drops `PrepareRound`'s round and returns an empty amendment (`internal/shedrecipe/entries_discussionwrite.go`).
  The preparation needs no seatengine field, and Scope's In list names only the two producer seams and the optional-values field, which D11 accounts for.
- Fixes this round: D3, D4 and D11 each state a bound; D8's bound is the one resting on a false premise (F2).
  Rejected-alternative narrative stays in the support log, and the requirements it carries are in the Decisions.
