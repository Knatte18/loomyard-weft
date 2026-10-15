---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D3 second bullet, D5)
    summary: The new loom halt on a repeated conflict names no way forward, which PATTERN-refusal-way-forward requires of every loom refusal and its spec row.
    fix: In D3's second bullet, replace "with the reason saying the same conflict survived a plan review" with a reason that also names the way forward, for example that the same conflict survived a plan review, and the way forward is to edit the named pending cards by hand so they match the tree, run `lyx webster rebaseline --card NN` naming each edited card, then `lyx loom resume`. Use whatever resume verb the loom section of contracts/specs/refusal-spec.md already names for a Webster stuck. In D5, state that the loom row for the repeated-conflict halt carries that same way forward. Add an Acceptance criteria clause saying the repeated-conflict Stuck reason names that way forward.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2, D3 Bound)
    summary: The conflict record outlives the refusal that wrote it within one Webster call, so a stuck from another cause after an earlier plan_conflict refusal is rerouted to plan review, and the Decision states no bound on this.
    fix: In D2, add that every begin-batch call removes a present conflict record first, under the mutate lock and before its re-resolution, and writes it again only on a plan_conflict refusal. A record then exists only when the latest begin-batch of this call refused with plan_conflict. In D3's Bound paragraph, add one sentence stating the remaining exposure and its bound - a master that goes stuck for another reason after that refusal is still rerouted, and the master stencil (D4) is what makes it stop at the refusal. Add an Acceptance criteria clause saying a begin-batch that refuses for a reason other than plan_conflict (quarry-unavailable, fingerprint mismatch) leaves no conflict record.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3 Bound, Open risks second bullet)
    summary: The claim that the Plan-Review ring keeps its own bounce budget omits that the Bouncer's hard cap is read from the round number, which carries over into the rerouted round N+1. With a small task's hard cap of 3, the rerouted round is usually at or past the cap, so a CONTINUE there is auto-accepted or escalated to a human instead of getting a fix round.
    fix: In D3's Bound paragraph, replace "the Plan-Review ring keeps its own bounce budget" with a statement that the reroute ends the Plan-Review episode, which resets Shed's bounce count, but the Bouncer's hard cap reads the round number, which continues as N+1. A rerouted round at or past the plan perch's hard cap therefore settles by the soft-cap rule. A CONTINUE with no design-wrong entry auto-accepts and goes back to Webster, where begin-batch checks again. A design-wrong CONTINUE escalates with cause hard-cap and the run waits on the operator. Add this consequence to the Open risks bullet that already names the inherited ledger and cluster exclusions.
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D3 first two bullets and Rationale)
    summary: The repeat check does not say what it compares. The strings Finding.Error renders embed card ids and card numbers, and the fix their own detail text suggests (moving a delete to a later card) changes those strings, so a surviving defect can read as a new conflict. One conflict then gets up to three plan reviews, not the one the Rationale claims.
    fix: In D3, state the comparison exactly. Either it is over the set of Finding.Error strings as D2 writes them (the order fixed by the record, the batch number and cause left out), and the Rationale then says a repeated conflict halts after one review only when the plan edit leaves the findings' card ids and numbers unchanged, with shed's reroute cap as the bound otherwise. Or it is over a key that survives a renumbering, such as each finding's Check plus the tree file and line or the referenced symbol it names, with the record carrying that key. Replace "after one plan review instead of three" in the Rationale with whichever bound the chosen comparison gives. Add an Acceptance criteria clause pinning that a reordered but otherwise equal set of findings counts as equal.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 says a re-begun first batch counts as moved when its fork landed work, but the begun-set test labels every re-begun batch tree_moved whether or not its fork landed anything, so the wording promises a distinction the rule never draws.
    fix: In D1, replace "a re-begun first batch whose fork landed work counts as moved" with "the begun set is DispatchScope's, computed before this begin-batch records its batch, so it holds the current batch on a re-begin but not on a first begin; a re-begun first batch is therefore tree_moved whether or not its fork landed work".
---

### [BLOCKING] Repeated-conflict halt has no way forward

**Location:** `_lyx/discussion/decision-record.md`, D3 second bullet and D5.

**Issue:** D3 adds a new halting Stuck from the Webster adapter when the conflict record's findings equal the last routed ones.
Its reason only says the same conflict survived a plan review.
`PATTERN-refusal-way-forward` binds loom: every refusal reachable through loom names its way forward in its message and has a row carrying it.
D5 adds a loom row for "the halt on a repeated one" but names no way forward for it either.
This also matters operationally: by the time this halt fires, the operator is the only party left to edit the plan, and the message is all they get.

### [MEDIUM] Conflict record can route a stuck of another cause

**Location:** `_lyx/discussion/decision-record.md`, D2 and D3's Bound paragraph.

**Issue:** D2 removes the record only at `websterengine.Run` entry and on a successful begin-batch.
Within one Webster call, a begin-batch refusal of another kind after a `plan_conflict` refusal leaves the earlier record in place.
That includes `ErrQuarryUnavailable`, which the Scope's Out list says must get no conflict record and no reroute, and a fingerprint refusal.
So does any stuck the master reports after that refusal.
The adapter then reroutes to plan review on a Go fact that no longer describes the stop.
D2's Rationale says routing must rest on a Go-written fact and not on the master's text.
As written, the fact holds only to the extent the master obeys D4's stop-at-once rule, and D3's Bound does not say so.

### [MEDIUM] Rerouted round inherits the hard cap by round number

**Location:** `_lyx/discussion/decision-record.md`, D3's Bound paragraph and the Open risks bullet on inherited ledger and exclusions.

**Issue:** The D3 premise itself holds.
`BurlerReviewProducer.Call` takes `highestCompleteRound` and, when that round carries a parsing verdict and ledger (`recordedVerdict`), spawns a reviewer for `highest + 1`.
A reroute also ends the Plan-Review episode (`internal/shedengine/doc.go`, "like a goto it ends the episode"), so Shed's bounce count starts fresh.
But `Bouncer.atHardCap` reads the round number alone, "since a goto resets that count while round numbers carry on", and `settleUndecided` opens the soft-cap window when `atHardCap(round)` holds.
An approved episode that took two or three rounds puts the rerouted review at round 3 or 4.
That is at or past the hard cap of 3 the orch stencil sets for small tasks.
There, a CONTINUE goes to `softCapRule`: it auto-accepts when no design-wrong entry is open, and escalates with cause hard-cap otherwise.
"The Plan-Review ring keeps its own bounce budget" is therefore not what the plan writer will meet.

### [MEDIUM] Repeat-check comparison is unspecified and unstable

**Location:** `_lyx/discussion/decision-record.md`, D3's first two bullets and its Rationale.

**Issue:** D2 records each finding "as `Finding.Error` renders it", and D3 compares "findings" with no word on the form being compared.
`Finding.Error` renders `check/card[severity]: detail`, where the card is the card id.
For `delete-before-reference`, the detail embeds both card numbers, the holder's card id and the tree `file:line` (`deleteOrderFindings` in `internal/planglyph/deleteorder.go`).
Ordering is deterministic, since files and cards are sorted, and the tree does not move during plan review, so the file and line are stable.
The card ids and numbers are not stable.
The finding's own advice, "move the delete to a card after card N", is a renumbering edit, and a partial fix of that kind leaves the defect in place under a new string.
The check then reads it as a new conflict and reroutes again, bounded only by shed's three-reroute cap.
The Rationale's "after one plan review instead of three" overstates it.

### [LOW] D1's re-begun-batch wording implies a landed-work test

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** `DispatchScope` puts a batch into `begun` whenever the state holds a record for it, terminal or not.
At begin-batch that record is written after the re-resolution, so the set excludes the current batch on a first begin and includes it on a re-begin, whatever its fork landed.
D1's "a re-begun first batch whose fork landed work counts as moved" reads as if a re-begun batch that landed nothing would count as `plan_defect`, and the rule never makes that distinction.

## Rejected

- D3 premise (focus 2): verified against `BurlerReviewProducer.Call` and `recordedVerdict`. A reroute after an approved episode spawns a reviewer for round N+1, and the Bouncer then resolves N+1 as unjudged and judges it, without hitting the clear-and-re-seed branch. Not a defect; the hard-cap consequence is F3.
- D3 bound against an edit that touches only pending cards but leaves the finding (focus 4): begin-batch refuses again, and the equal-findings arm halts, subject to F4's comparison. Rerouting every blocking class bypasses no guard. Each edit still passes `ValidatePlan` (`done-card-edited`), the Bouncer judge and `ErrRebaselineCardSetChanged`, and begin-batch re-runs the full dispatch check on re-entry.
- D6 and D7 attack surface (focus 6): each Decision states its bound. A caller with a terminal stdin but redirected stdout reaches the editor. D7 catches an empty result, and an unchanged non-empty file is restored byte-for-byte, so nothing is lost. D7 states that a layer can no longer be emptied through the editor, and how to clear one instead.
- Relocation check (focus 7): D3's route narration and the `sheddriver` sentence state consequences the plan writer needs for tests and docs. No passage in Decisions or Open risks is deliberation narrative that `support-log.md` should hold instead.
