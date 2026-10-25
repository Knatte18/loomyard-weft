```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions that act on the same state, and checked the criteria against the decisions they touch.

**Pairs checked**
- D1 × D6/D9/D10 (`DecisionNoteTitle` moves; the lenses, the writer and the darn writer read decision notes): held.
- D2 × D9 (both edit `loom-template-discussion.md`: D2 removes the parent-review text, D9 adds the parent-only rule): held.
- D2 × D8 (the Discussion-Write gate goes, and the Bouncer escalation is the only parent route): held.
- D2 × D4 (`internal/parentreview` is deleted while reports keep reading its records through a parser inside `reportengine`): held.
- D3 × D11 (retired loom keys are dropped and landing/gate keys renamed, both by reconcile and read-time migration): held.
- D4 × D11 (reports keep filing an archived `Merge-In` row as a retired name): held.
- D5 × D6 (D5 says D6's reads fit the fork's 6-call cap): broken, F1.
- D6 × D7 (D7 defines `decision` as D6's cases, while D6's own both-lenses rule sends an ordinary misalignment to the fixer): broken, F3.
- D7 × D8 (a narrow class, with any-severity escalation): held.
- D7 × D9 (an Open risks line a decision note answers becomes `consistency`): held.
- D8 "never a settled generation" × D8 accept path, and × the existing settled-entry/new-generation paths: broken, F2.
- D8 arbiter refusal × D8 hard-cap handling (`retireContinue` re-escalates with cause `hard-cap`): broken, F4.
- D8 Bouncer switch × D8 fix-row silence (two rows, one parent message): broken, F5.
- D8 × `discussion_auto_accept` and the soft-cap rule (D8 runs first): held.
- D8 × Scope Out (the Plan and Code perches are unchanged, the key is set only on loom's Discussion-Bouncer): held.
- D9 × D6/D8 (the writer's Open risks line is raised as `decision` and escalated): held.
- D10 × D8 (darn has no Discussion-Bouncer; darn's Code perch keeps the notice): held.
- D10 internal (the in-record Decision, then `lyx loom decision add` after the commit): held.
- D11 × Constraint "D2 and D11 change no row position": held.
- Acceptance criteria × D8: the criterion "A re-call over a round escalated after a CONVERGED verdict returns the same Awaiting" has no counterpart for a re-entry after the accept is settled (F2). The criterion "at or past the hard cap offers accept only" says nothing about the cause turning into `hard-cap` (F4). The other criteria held against their decisions.

**Findings**

- **F1 — MEDIUM, design.** Location: D5's claim "Each fork has 6 tool calls … which D6's reads fit", against D6.
  - `goal-scope` reads "every `Decision for <slug>:` note for it", and the number of notes has no bound. Finding the notes also needs `lyx board list`, which `goal-scope`'s reads do not include, and then a `lyx board get` per note.
  - `board-fit` uses all 6 calls on its own reads: `lyx board list`, `lyx loom peers`, then up to four. The both-lenses rule still requires it to honour decision notes ("A board note titled `Decision for <slug>:` … wins"), which takes further `lyx board get` calls past the cap.
  - So D5's fit claim fails against D6. A fork either goes over the cap or skips the notes, and then raises a `decision` finding the parent has already answered, which D8 then escalates.
  - Evidence: `burler-cluster-fan.md:17` ("at most 6 tool calls"); decision-record.md lines 69 and 74–85.
- **F2 — MEDIUM, design.** Location: D8, second bullet ("A round so escalated is never a settled generation at Call entry … never the settled-entry harvest, clear or new generation, whatever `lyx loom decision add` committed meanwhile").
  - The rule has no exception for an accept that has already been settled. After `acceptRound` settles the round (`settleCirclingAccept` then `finishSettle`), a later Call reaches the round again through the recorded-decision path. That path has `case settled:` degrade "already settled; way forward: run `lyx loom start`, which re-enters the segment" (bouncer.go `settleUnconverged`). Re-entering lands on the same degrade, so the run loops.
  - Two ways in: a crash between the settle and the driver persisting Done; and a later amendment `lyx loom decision add` that needs the settled-entry new-generation path (cases 1–3 of the `settledGeneration` doc).
  - Today `settledGeneration` returns true for a settled accept, and for any CONVERGED round.
  - D8 should end the exclusion once the round's accept is settled. A criterion for re-entry after a settled `decision` accept is missing.
  - Evidence: `internal/shedadapters/bouncer.go:833-843` (`settledGeneration`), and `settleUnconverged`'s `case settled` branch (around line 1199).
- **F3 — LOW, consistency.** Location: D6 `board-fit`'s list of parent questions, against D6's "An ordinary misalignment the writer could settle is an ordinary finding for the fixer" and D7's "A choice the writer can make is never `decision`".
  - D6 makes "a landed parent commit that already makes or contradicts one of the record's decisions" a parent question in every case.
  - When the commit already makes the decision, the writer can usually settle it by dropping or citing that decision. D6 and D7 then send the same case two ways.
  - The bullet should name only the contradiction, or the case where the writer cannot settle it.
  - Evidence: decision-record.md lines 80, 89 and 98.
- **F4 — LOW, design.** Location: D8 "`internal/arbiterstop` refuses a `decision` escalation", against D8 "at or past it, accept only, as every cause does" and the existing hard-cap path.
  - A `continue` recorded below the hard cap on a `decision` escalation, where the round is at or past the cap at settle time (for example after `review_hard_max_bounces` is lowered), takes `settleUnconverged`'s `retireContinue` → `escalate(round, EscalationHardCap)`.
  - That moves aside the `decision` escalation record and writes a `hard-cap` one, and `arbiterstop.Eligibility` accepts `hard-cap`.
  - The arbiter can then accept a round whose `decision` finding the parent never answered.
  - D8's arbiter bound needs the cause kept as `decision`, or the refusal keyed on the round's `decision` findings rather than on the cause.
  - Evidence: bouncer.go `settleUnconverged` (the `CirclingContinue` + `atHardCap` branch) and `retireContinue`; `internal/arbiterstop/arbiterstop.go:77` (`hard-cap` and `circling` eligible).
- **F5 — LOW, consistency.** Location: D8's last bullet (Discussion-Burler-Fix "posts no notice of its own") and the Constraint "`PATTERN-shed-recipe-registry`: D8's switch is looked up by key".
  - D8 names one key, on the Discussion-Bouncer only. It never says how the separate fix row learns to stay silent.
  - Two independent switches can disagree. With the fix row silenced and the Bouncer key unset, the parent gets no message at all, and the record leaves that pairing unstated.
  - D8 should name the fix row's key, or say that the fix row reads the Bouncer's key.
  - Evidence: decision-record.md lines 105, 114 and 169.

**Tool calls**
1. `sed -n 51,84p internal/shedadapters/circling.go` plus a grep of `internal/loomcli/circling*`: checked whether `circling continue` at the hard cap is refused, and how the decision cause is recorded (for F4). My context held the Bouncer's settle path but not the verb's refusal.
