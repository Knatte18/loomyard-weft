---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D6, Open risks)
    summary: The all-Prosa exemption lets a Prosa card edit any Go file or whole package unreviewed by Plan-Review, and the record names only the stencil case and never states the bound
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D6, D7)
    summary: The skip seam trusts a stored class in record.json that any agent can write (the rework dir is swept by every status commit) and never re-checks the live cards it skips
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D9, D10, D15)
    summary: D9 and D10 trigger on any committed round, not on the round that produced the live generation, so a goto Plan-Write after a rework round gets the wrong answer key and review range
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D2, D6, D14)
    summary: D14 demands archive-done vs archive-interrupted be told apart but no Decision names the marker; record.json timing differs across D6 and the plan-writer note, and roundFor today allocates a new round for a record-less directory
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (D1)
    summary: D1 places the Plan-Review and Webster-Review run directories under _lyx/loom/reviews/, but LoomReviewsDir is _lyx/reviews/
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (Constraints, Open risks)
    summary: The append-only removal bound says built cards cannot be rewritten by construction, but nothing stops the rework session editing the archived generation it reads, which Go then commits
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md (D12 Rationale)
    summary: D12's rationale opens with refusal-audit history, deliberation narrative that belongs in support-log.md
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md (Auto-mode assumptions)
    summary: Most Auto-mode assumption lines restate a Decision in full instead of pointing at it, so the two copies can drift
---

### [MEDIUM] The Prosa exemption is unbounded on code files

**Location:** `decision-record.md` D6 and Open risks.

**Issue:** D6 exempts a generation when every card's every type label is `Prosa`.
`contracts/specs/loom-plan-spec.md` defines `Prosa` as "file(s), no symbol target" with no mechanical check, and reserves it for documentation only in prose (line 303).
The only mechanical rule on a `Prosa` group is `prosa-symbol-target` (`internal/planparser/validate.go`, `checkProsaSymbolTarget`): under a glyph-enabled language it admits any file self glyph or whole-package self glyph, so `internal/loomshed/prrework.go#` or `internal/loomshed#` pass.
The spec's own example card 7 targets `doc.go` under `Prosa`.
So an exempt generation can carry a card that rewrites any Go file or package and skip Plan-Review.
Open risks names only the stencil case, and neither D6 nor Open risks says what does bound it (Webster-Review still reviews the resulting code against the card, and the operator reviews the PR).
By D12's own rule, an exemption whose reach and bound are unstated is a finding; D6's own principle is to over-cover rather than under-cover.

**Fix:** Make D6's exempt class require every `Prosa` target to be a non-source file: a `Prosa` target that is a whole-package self glyph, or a file self glyph whose extension is a source file of the plan's `language:`, makes the generation `required`.
Restate the Open risks entry as the residual this leaves (a `Prosa` card on a stencil or other non-source file that agents follow as rules), bounded by Webster-Review over the resulting diff and the operator's PR review.

### [MEDIUM] The skip seam trusts a stored class from a file agents can write

**Location:** `decision-record.md` D6 and D7.

**Issue:** D7's seam skips review when the highest committed round's `record.json` says `exempt` and its `first_card` matches the live overview's.
Neither input is Go-only: the live overview's `first_card` is written by the rework session, and `_lyx/loom/rework/` sits under `LoomDurableDir`, which `statusCommitPathspec` (`internal/loomcli/wiring.go`) commits on every status transition, so a `record.json` any agent writes under it is committed by Go.
The support log's own rejected alternative says the skip signal must be something only Go writes.
Separately, the seam reads a class computed at classification time and never re-checks the cards it is about to skip; nothing in one run changes the cards between PR-Rework's commit and Plan-Bouncer, but the accepted operator `goto Plan-Bouncer` (Open risks) can arrive after the live cards changed, and still skips on the stale class.
The record does not say who writes either input or what a stale or forged record can let through.

**Fix:** Have the skip seam re-classify the live committed cards with the same rule D6 uses and skip only when that live class is `exempt`, the round record says `exempt`, and the `first_card` values match.
State the bound in D7: a stale or forged record can at most skip review of a generation whose live cards are themselves exempt.

### [MEDIUM] D9 and D10 key on the wrong round after a goto Plan-Write

**Location:** `decision-record.md` D9, D10 and D15.

**Issue:** D7 ties the skip to the round whose `first_card` matches the live overview, but D9 (answer key gains the highest round's `findings.md`) and D10 (review range starts at the highest round's `head_sha`) trigger whenever `_lyx/loom/rework/` holds any committed round.
After a rework round, an operator `goto Plan-Write` writes a fresh plan from the decision record (no `first_card`, so 1).
Plan-Review would then demand that this plan cover round N's findings, and Webster-Review would review only commits after round N's `head_sha` against a plan that describes the whole task.
Plan-Write's rotation also moves the live rework generation into `_lyx/plan/archive-*/`, a retired generation inside `_lyx/plan/`, which D15's invariant as worded forbids.

**Fix:** Define the live generation's round once: the highest committed round whose `record.json` `first_card` equals the live overview's `first_card`.
D7, D9 and D10 all key on it; with no such round, the live plan is treated as generation 0 (decision record only, merge-base range, real review).
Word D15 so Plan-Write's own `archive-*/` rotation is the one exception the invariant names.

### [MEDIUM] No Decision names the archive-completion marker

**Location:** `decision-record.md` D2, D6 and D14, and Notes for the plan writer.

**Issue:** D14 requires a resumed PR-Rework never to move a file the session wrote, which needs a marker that tells "archive done, session begun" from "archive interrupted".
No Decision names it.
D6 writes `record.json` after the session and gate pass; the plan-writer note suggests writing it last in the archive step, before the session.
`roundFor` (`internal/loomshed/prrework.go`) today matches a round by the working-tree `record.json` and returns highest-plus-one for a directory without one, so a crash mid-archive would allocate a new round and orphan the half-archived one.
The requirement sits only in the optional notes, which are a head start, not a contract.

**Fix:** Put the marker in D14: the archive step writes `record.json` (rejection identity and `first_card`) last, after every move; a round directory whose record matches the pending rejection means the archive is done and is never re-run; the highest round directory with no `record.json` is the pending rejection's interrupted archive and is resumed, never a new round.
D6 then adds the class to that record after the gate passes.
D2 and the note follow.

### [LOW] Wrong review run-directory path in D1

**Location:** `decision-record.md` D1.

**Issue:** D1 says the Plan-Review and Webster-Review run directories are under `_lyx/loom/reviews/`.
`loomengine.LoomReviewsDirRel` is `_lyx/reviews`, and the run directories are `_lyx/reviews/plan` and `_lyx/reviews/webster`.

**Fix:** Correct the path in D1.

### [LOW] The append-only removal bound overstates "by construction"

**Location:** `decision-record.md` Constraints (Fabric Git) and the last Open risks entry.

**Issue:** The Open risks entry says D1 makes rewriting built cards impossible by construction.
That holds for `_lyx/plan/`, but the rework session reads the archived generation under `round-<N>/prior-generation/` and nothing forbids it editing those files; the Fabric Git constraint forbids only committing, moving or deleting the archive.
Go's commit records the archive as the session left it, and D11 tells Describe the archived `summary.md` files.

**Fix:** State in Constraints that the archive is read-only to the rework session, and in the Open risks entry that the residual is an edit to the archive, visible in the round commit's rename diff against the moved-from paths.

### [NIT] Deliberation history in D12's rationale

**Location:** `decision-record.md` D12 Rationale.

**Issue:** "refusal-audit's own review wrote ... and nothing acted on it" is history the plan writer does not act on.
The second clause, where the question has to be asked, carries the reason.

**Fix:** Move the refusal-audit history into `support-log.md`'s Interview 9 and keep the second clause.

### [NIT] Auto-mode assumptions restate Decisions

**Location:** `decision-record.md` Auto-mode assumptions.

**Issue:** Each line restates a Decision (D6, D3, D1, D1, D10, D11, D12, D13, Scope Out) in its own words, a second copy that can drift from the first.
What a line adds is that the pick interprets the board rather than an operator answer; the restated content adds nothing for the plan writer.

**Fix:** Reduce each line to the Decision it points at plus the board interpretation it makes.

## Focus verdicts

- Focus 1 (Prosa on non-documentation files): nothing mechanical stops it; F1.
- Focus 2 (attack surface of the new seams): the skip seam is F2; the append-only deletion is bounded for `_lyx/plan/` by D1, with the residual in F6; the exempt class in `record.json` is F2's stored-class half.
- Focus 3 (D9/D10 vs D7 after goto Plan-Write): F3.
- Focus 4 (crash-resume marker and `record.json` timing): F4.
- Focus 5 (relocation): F7 and F8. The other Rationale lines and Open risks carry reasons or residuals the plan writer acts on and stay.
