---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (design_prefix refusal)
    summary: Refusing only a design_prefix equal to the record prefix still lets the render write over and later delete a record file, since any prefix that is a proper prefix of decision- or starts with decision- renders a design file onto a decision-<slug>.md name.
    fix: In D1, replace the equality refusal with a refusal of every design_prefix that is a prefix of the record prefix decision- (the empty prefix included) or that starts with decision-, keep the stated way forward, and widen the D1 acceptance criterion to name an empty prefix and a proper prefix such as dec beside the equal one. Update Q11 in support-log.md to match.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 (rename bullet)
    summary: D4 moves the record file on rename "in the same write" without saying when; a move inside the mutation survives a rename that checkNewSlugs refuses after fn, leaving the old slug flagged with no file, and breaks rename.go's guarantee that a refused rename writes nothing.
    fix: In D4's rename bullet, state that the move runs after the save, in the board's afterSave step under the board lock, as D4's removals do, so a rename refused by any post-mutation check moves nothing; add to the D4 acceptance criterion that a refused rename leaves the record file at the old name.
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks; D7, D8, D9
    summary: The Open risks line asks about a coordination D7's guard already settles (whichever task lands second adds the key to cartoon's Bouncer), while the cross-run question that remains is unraised, namely that D8 and D9 judge the approval row from the recipe's own rows, which a shed-sub-run-block recipe running cartoon inline as a block does not hold.
    fix: "Replace the Open risks line with the unsettled question for the parent: whether a recipe that runs cartoon through shed-sub-run-block's block must be reached by D8's copy and D9's verb, and if so which task defines how the passed-approval test reads a row inside a block. Drop the current line, since D7's guard settles it."
  - id: F4
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 and Auto-mode assumptions (older binary); support-log.md Turn 4
    summary: An older binary's rename carries discussed onto the new slug as an unknown key while the file stays at the old name, so a flag without a file is reachable without hand deletion, contrary to D2's invariant claim and the support log's rejection of the D10 finding.
    fix: In the Auto-mode assumption on older binaries, add that an older binary's rename leaves the new slug flagged with no file, which DecisionRecord answers as not found, until the new binary runs; in D10 give --record a refusal with its way forward for a flagged entry whose file is missing (re-store with lym loom decision store); in support-log.md change the rejected D10 item to fixed.
  - id: F5
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 (merge bullet) and D3
    summary: A merge that removes slug X and upserts X again keeps X's record file under D4's net-change rule, but MergeTasks rebuilds X through NewTask, so D3 drops or refuses the carried discussed value and the entry ends unflagged with an unreadable record.
    fix: In D4's merge bullet, state that a merge which upserts a slug it removes keeps that entry's discussed value along with its file, and extend D3 to say the stored value it judges a payload against is the removed entry's value in that case; add the kept flag to the D4 acceptance criterion's merge case.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 (Bound line)
    summary: D2 bounds the run-lock bypass by saying only Go callers reach StoreDecisionRecord, but D9 makes it reachable by hand from the parent orch, the operator's shell and the arbiter strand.
    fix: Rewrite D2's Bound to name the routes that reach the method, the Discussion-Bouncer approve seam (D6), decision add (D8) and the hand verb lym loom decision store (D9), and to point to D9's bound for what the hand route may store.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D8, D9 against D2
    summary: A decision add whose entry is already done fails the copy and names lym loom decision store as the way forward, but D9 calls the same StoreDecisionRecord that D2 refuses on a done entry, and neither D8 nor D9 says what a post-landing call meets.
    fix: In D8, state that a copy refused because the entry is done is reported as record_store_error naming that a landed entry keeps the record it last stored, without pointing at decision store; in D9, add the done-entry refusal with that same wording; add the done-entry case to the D8 and D9 acceptance criteria.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6, D8, D9 (goto past Discussion-Bouncer)
    summary: After a goto past Discussion-Bouncer, D8 copies the never-approved record on the next decision add without saying so, D6's bound reads as if only D9 stores, and D9's bound claims it flags only where an approved record exists before conceding the goto case.
    fix: Add to D8's Bound that a run a goto moved past Discussion-Bouncer counts as past, so decision add copies its record, as the parent's goto decided; in D6's Bound name both D8 and D9 as storing such a run; in D9's Bound drop "only where the run's approved record exists" and say it stores only past the approval as D8 judges it, a goto past counting as past.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 (carry-over sentence) and Goal
    summary: D8 says carry-over blocks later review segments write reach the board copy, and the Goal says decision add keeps the copy current, but a Plan-Review or Code-Review settle commits its carry-over into the record with no store, so the board copy lacks it until a later decision add or a hand store.
    fix: In D8, replace the carry-over sentence with the bound that the board copy is refreshed only at Discussion approval, on decision add and by lym loom decision store, so a later segment's carry-over reaches it only through one of those.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 (after its commit succeeds)
    summary: decision add's existing failed-commit refusal sends the caller to lym fabric commit, which lands the record but never copies it, and D8 is silent on whether the landed-despite-error path copies.
    fix: In D8, state that the landed-despite-error path copies as a successful commit does, and that the failed-commit refusal adds lym loom decision store as the step after lym fabric commit lands the record; add the failed-commit case to the D8 acceptance criterion.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D9 (caller rule)
    summary: decision add admits the arbiter only through --by arbiter plus its arbiter-strand check, and D9 has no --by flag, so judging the arbiter as decision add judges it is undefined.
    fix: In D9, state that the arbiter is admitted by the run's arbiter-strand check alone, with no label, and that the parent orch and the operator's unnamed shell are judged by decision add's parent and unnamed-shell rules.
---

### [MEDIUM] D1's prefix refusal leaves record files open to the render

**Location:** `_lym/discussion/decision-record.md` D1.
**Issue:** `renderDesigns` names a design file `<design_prefix><slug>.md` (`internal/boardengine/render.go:795`), and `RenderToDisk` writes every rendered file, then deletes names that left its manifest (`render.go:43-63`).
`design_prefix` is free-form (`internal/boardengine/template.yaml:2`, no validation in `config.go`).
With `design_prefix: dec` and an entry slugged `ision-foo`, or an empty prefix and an entry slugged `decision-foo`, the render overwrites entry `foo`'s record with design-doc text and later deletes it.
Refusing only the equal prefix does not uphold D1's claim that the render never owns a record file.
Found by bound-breaker, walkthrough, existing-paths and the handler.

### [MEDIUM] D4's rename move has no stated point in the write

**Location:** `_lym/discussion/decision-record.md` D4.
**Issue:** `checkNewSlugs` runs on the renamed slug after the mutation (`internal/boardengine/board.go:144`).
`rename.go`'s file header promises that a refused rename writes nothing.
A file move made inside the mutation, as D2's file-first store is, outlives such a refusal.
The result is the old slug flagged with its file gone, the flag-without-file state D2 sets out to prevent.

### [MEDIUM] The Open risk names a settled coordination and misses the open one

**Location:** `_lym/discussion/decision-record.md` Open risks, D7–D9.
**Issue:** D7's guard test already settles the stated line: whichever task lands second fails until cartoon's Discussion-Bouncer carries the key.
So that line, read alone, would be a consistency finding to drop.
The real cross-run question is elsewhere.
Task `shed-sub-run-block` (board entry, status Discussion-Write) builds a recipe that runs `cartoon` inline as a block, then tapestry's Plan and build rows.
D8 and D9 find the approval row in "the recipe's rows" and judge "past" by the recipe's row order.
That recipe holds no DiscussionWrite row of its own, so D7's guard passes over it trivially, and D8 never copies for it.
How a row inside a block is named is still open in that run's Discussion.
Who defines the passed-approval test for a block is therefore a design conflict with another run, which only the parent settles.

### [LOW] An older binary's rename leaves a flag without a file

**Location:** D2, Auto-mode assumptions, `support-log.md` Turn 4.
**Issue:** The Constraints line says an older binary keeps `discussed` as an unknown key and writes it back.
So an old binary's rename of `a` to `b` flags `b` while the file stays `decision-a.md`.
The support log rejected a D10 finding on the claim that this state needs a hand deletion, and that claim is false.

### [LOW] A merge that removes and re-upserts a slug drops the flag

**Location:** D4 merge bullet, D3.
**Issue:** `Store.MergeTasks` drops the removed slug before it builds the upsert, so the upsert goes through `NewTask`, not `ApplyPatch` (`internal/boardengine/store.go:1005-1060`).
D4 keeps the file for that slug.
D3 has no answer for a payload that carries `discussed: true` on what is now a new entry, so the entry loses its flag, and `DecisionRecord` reports its record as not found.

### [LOW] D2's bound omits D9's hand route

**Location:** D2 Bound.
**Issue:** "Only Go callers reach the method" leaves out the D9 verb, which the parent orch can call.
The run lock exists to restrain the parent orch.
D9's own bound is what limits the hand route.
Found by interactions, bound-breaker and the handler.

### [LOW] D8 and D9 point a landed run at a refused way forward

**Location:** D8, D9 against D2.
**Issue:** D2 refuses a done entry (premise verified: the ledger's `Ended` is `isDone`, `internal/boardengine/replay.go:252`, and a by-entry commit whose runs have all ended is set aside, `internal/fabricengine/replay.go:635-695`).
D8's `record_store_error` names `decision store`, which meets the same refusal.
Found by interactions, bound-breaker and the handler.

### [LOW] D6, D8 and D9 disagree on the goto-past case

**Location:** D6, D8, D9 Bounds.
**Issue:** D9 concedes that a goto past Discussion-Bouncer counts as past.
D8 silently copies in that state.
D6's bound names only D9.
D9's first clause claims that an approved record always exists.
Found by interactions, bound-breaker and the handler.

### [LOW] D8 overstates how carry-over reaches the board copy

**Location:** D8, Goal.
**Issue:** The `CarryOver` seam commits the record alone on every converged settle of any segment (`internal/loomcli/wiring.go` CarryOver wiring, `internal/shedadapters/bouncer.go:1164`).
No store runs on those settles.

### [LOW] D8 misses decision add's failed-commit paths

**Location:** D8.
**Issue:** The refusal text `decisionWayFabricCommit` and the `afterFailedCommit` landed path (`internal/loomcli/decision.go:579-605, 780-788`) are not covered by "after its commit succeeds".

### [LOW] D9's arbiter test is undefined

**Location:** D9.
**Issue:** `decision add` admits the arbiter through the `--by arbiter` label and the arbiter-strand check together (`internal/loomcli/decision.go:8-9, 645-648`).
D9 carries no label.

## Rejected

- Walkthrough's and interactions' note that the D6 acceptance criterion omits the skipped-review path: no Discussion-Bouncer row carries `skip_seam` in `contracts/recipes/tapestry-recipe.yaml`, so the path is unreachable for this row and no criterion is owed.
- The focus premises `scopeChanged` never compares `discussed`, the run ledger sets aside ended commits, `afterSave` runs under the board lock, and the design-file rule fits a record file: each was checked against the code (`runlock.go:189-213`, `fabricengine/replay.go:635-695`, `board.go:156`, `boardengine/replay.go:532-566`) and holds, so none is a finding.
- Leanness: no Decision, Constraint or Auto-mode assumption carries deliberation narrative to relocate, and every acceptance bullet tests a behaviour its Decision states; no relocation or exclusion finding.
