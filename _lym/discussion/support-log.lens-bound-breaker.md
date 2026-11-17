```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and ordering promises with concrete counterexamples. Two claims break outright and three are only partly held.

**BLOCKING — D6, check order.** The claims that the stalemate check runs before the judge and that "converged, decision and hard-cap paths ... win over it" break.
- Evidence: in `internal/shedadapters/bouncer.go`, the hard-cap escalation and the soft-cap window are decided in `settleUnconverged`, which is reached only after the judge (`rejudge` ~L1186–1209, then `atHardCap` → `EscalationHardCap` at ~L1260). Converged is the judge's own `CONVERGED` verdict, not a Go check before it.
- Counterexample 1: with `review_stalemate_rounds: 2`, round 2's review holds only MEDIUM findings and the fixer disputes them all, leaving the target unchanged. The stalemate check fires before the judge, so the run escalates `stalemate` to the parent on a segment the judge would have settled `CONVERGED`.
- Counterexample 2: at the hard-cap round with an unchanged target, the run escalates `stalemate`, not `hard-cap`.
- The fix: the record must state a position that matches today's code. Either run the stalemate check inside the unconverged path (after a `CONTINUE` and before the soft-cap and hard-cap rules), or say which way each pair resolves.

**MEDIUM — D1 and its acceptance criterion, rulings.** "a `decision`-class finding ... one a ruling reached needs a `ruling` entry" cannot be checked by Go.
- Evidence: a ruling reaches the fixer as parent creel mail or a decision notice (`internal/burlerengine/doc.go` ~L158–162). Neither `shedadapters` nor `burlerengine` records which findings a ruling reached; `sendDecisionFindings` only posts outward.
- Counterexample: a ruling arrives and the fixer omits the `ruling` entry. The fix check cannot tell, so the criterion is untestable as stated.
- The fix: make a `ruling` entry on a decision-class id optional (allowed, never required), or name the Go record that would make it checkable.

**MEDIUM — D2 and D3, unruled decision findings.** Excluding unruled decision-class findings from the prior-findings set lets them vanish.
- Path: a segment whose Bouncer does not escalate decisions (`decisionsEscalated` false) mails a round-N decision finding to the parent once.
- Round N+1 is incremental. The finding is neither in `prior` nor carried. Raising it again would make it a new finding, refused unless it is BLOCKING and on a changed file.
- The open decision therefore drops out of every later review, and the Bouncer's decision path never sees it again.

**MEDIUM — D4, which rounds are full.** "Full" covers only a segment's first round and an amendment's first round.
- Evidence: the Bouncer also starts full generations (`FullGeneration`, "a changed subject start[s] a full generation", `bouncer.go` ~L211) when the subject is new rather than amended.
- Counterexample: a goto or re-bounce that rewrites the subject starts a new full generation. Its first round would still be incremental under D4, so a new subject gets a diff-scoped, BLOCKING-only review.

**MEDIUM — D4, the location check.** "its `location` path is a file the changes file touches" has no stated path form.
- Overlay targets are `_lym` documents. Locations there are often section references (for example `decision-record.md D3`) or absolute paths, while Go builds its diff from `.lym` copies.
- Counterexample: a legitimate BLOCKING finding with `location: _lym/discussion/decision-record.md#D3`, an absolute path, or a section reference is refused or let through depending on unstated normalization.

**LOW — D5, untracked files in the source snapshot.** The working-tree id hashes every untracked file that is not ignored, such as test binaries, coverage output or fixer scratch.
- Stalemate errs safe: a junk file only makes a round read as changed.
- D4's changes file, however, lists these junk files as touched, which widens the scope for new BLOCKING findings.

**Claims that held:**
- D5/D6: a missing snapshot record counts as changed, and a missing cache makes the next round full. The record is tracked and the cache is not, so the two never conflict.
- Overlay hash stability between rounds: Go writes the decision record (carry-over) only at settle and the support-log headers only at creation, so neither changes the target's bytes between rounds (`internal/loomshed/recordwrite.go`; `CarryOver` doc at `bouncer.go` ~L172–174).
- In-flight run across the binary change: an old round has no snapshot, so the next round is full and the check refuses `prior`/`carries` there. This is consistent.
- D6's bound that the escalation is Awaiting and not counted against the budget matches how the other escalations work.
- D3's severity cap and D4's BLOCKING-only rule: a reviewer can mark a prior finding resolved and raise a new BLOCKING finding on the changed file. This is allowed and does not break the bound.

**Tool calls:**
1. grep for Go writers of `## Review rounds`, the `CarryOver` timing in `bouncer.go`, and ruling records. My context lacked whether Go writes target bytes between rounds and how rulings are tracked.
2. grep for ruling and amendment handling in `burlerengine` and `burlerreview.go`, plus `sendDecisionFindings`. My context lacked how rulings reach the fixer and whether a review knows it opens an amendment.
3. grep for the Bouncer's check order (judge, `atHardCap`, `settleUnconverged`, amendment). My context lacked where hard-cap and converged are decided relative to the judge.
