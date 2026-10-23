---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/29-finalize-gate-and-fixer.md:50
    summary: "Card 29 prescribes the over-cap resume over a failing gate, and the no-attempts-left branch of it, but its test names no row that reaches either; card 26 pins the same paths at Merge-In"
    fix: >-
      In card 29's `TestFinalize_GateAndFixer` paragraph (line 50), add three rows beside the existing row that resumes an over-cap halt over a passing gate, all with `RerouteReviewed` false.
      Row 1: a one-turn fix that edits a `_test.go` file under a Go matcher halts over-cap with `Spent` 1 of the default 2 and `over-cap` noted; its resume over a failing gate spawns the `finalize` fixer on the one attempt left, `LineChange` is called from the `Baseline` the resume wrote afresh at the HEAD `gateAndFix` starts from, and the note is cleared in the same record write.
      Row 2: the same entry where the fixer on the resume turn answers `escalate`, then a second resume calls `LineChange` from the `Baseline` the first resume wrote, never from the HEAD the second resume starts from, and reads `Halt` as empty.
      Row 3: a two-turn fix that spends both attempts and halts over-cap; its resume over a failing gate spawns no fixer and halts with `LandingStopPrefix` naming the log, as a spent cap does where the way back reaches no review.
      These mirror the last rows of card 26's `TestMergeIn_Fixer` paragraph.
      The fix only adds test text to one card paragraph and adds or changes no target, so no `validate-plan --overlay` run constrains it.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/35-driver-hands-back-landing-halts.md:26
    summary: Cards 35 and 36 plan one test row per kind of landing halt, but both code paths read only the marked predicate that TestHandsBack in card 34 already pins per kind
    fix: >-
      In card 35 (line 26) change the `TestDecideDrive` rows from "a marked blocked stop at each kind of landing halt" to one marked blocked stop row, keeping the changed-build-with-escalation-pending row and the Webster reroute-cap row, since `decideDrive` reads only the `Stop.HandBack` boolean.
      In card 36 (line 23) change `TestCheck_RefusesLandingHalts` from five per-kind rows to one landing-halt row (any `LandingStopPrefix` reason at Finalize) plus the two eligibility rows it already lists, since `Check` sets `LandingHalt` only through `landingshed.HandsBack`.
      Each kind's reason shape stays pinned once, in card 34's `TestHandsBack`, which already lists a landing-stop reason at both rows, the undecodable-record halt, the lock re-check halt and the reroute-cap reasons.
      The fix edits test text only and adds or changes no target.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/40-idle-release-keeps-finalize.md:27
    summary: Card 40 downgrades both idle-release paths to keep the lock while Finalize runs or a verify marker is live, and keeps it on a status read error, but its Intent states no bound on that hold
    fix: >-
      In card 40's Intent (line 27 and the `KeepsLock` sentence after it) add one sentence stating what bounds the kept lock, since the idle release is the safety valve for a stalled loop and this card makes it skip a running Finalize.
      The bounds to name are the ones the plan already carries elsewhere, so no new mechanism is introduced: a dead owner process is taken over at the next acquire (card 27), a hung fixer turn ends at the session bound of `fixer_timeout_min` times the turns left (card 11), a marker is live only while its verify process lives, and an operator's `lyx batten lock release` still frees the record.
      The same sentence states that a status read error keeps the lock for as long as the read fails, with the once-per-episode Warn as the signal, so the hold is visibly unbounded only for an unreadable status file.
      The fix edits Intent text only and adds or changes no target.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/07-fixer-session-table.md:30
    summary: Card 7 says merge-in mode always carries an aligned tip but lists AlignedTip as an Optional table value without saying the optionality is finalize-mode only, so an empty tip in merge-in mode would render silently
    fix: >-
      In card 7 (line 30) state that `FixerTable` lists `aligned_tip` among the table's `Optional` values only when `Mode` is `finalize`, so that seatengine's blank-value check refuses an empty `AlignedTip` in `merge-in` mode by name, and `PlanDir` and `FailureRecordPath` stay Optional in both modes.
      In the test paragraph (line 33) add the matching row: a `merge-in` table built with an empty `AlignedTip` is an error naming it, while the same field empty in `finalize` mode renders empty.
      This turns the stated property "merge-in mode always carries one", which card 26's first-Merge-In row pins from the caller side, into one the table enforces, so a caller bug in the fork-point rule cannot hand the stencil an empty start tip.
      The fix edits text only and adds or changes no target.
---

### [MEDIUM] Card 29's test names no row for the over-cap resume over a failing gate

**Location:** `_lyx/plan/29-finalize-gate-and-fixer.md:50`
**Issue:** Card 29 prescribes three outcomes for a resumed Finalize whose record notes `over-cap`: over a passing gate it lands without judging the change, over a failing gate it runs the fixer on the attempts left, and over a failing gate with none left it reroutes or halts as a spent cap does.
The test paragraph pins only the first.
The other two are branches the card prescribes that no named input reaches, and the invariant that matters for the focus question, that the resume writing `Baseline` afresh also clears the note so a later `escalate` halt leaves `Halt` empty and the next resume keeps that fresh `Baseline`, is pinned at Merge-In (the last rows of card 26's `TestMergeIn_Fixer`) and nowhere at Finalize.
Left unpinned, a Finalize that left the `over-cap` note set would rewrite `Baseline` on every later resume and let an escalated turn's commits escape the cap, the exact failure D4 and D10 bound.
Tracing the plan text itself, the resume is liftable at both rows over a passing gate (cards 26 and 29 agree), and over a failing gate at Merge-In; at Finalize the failing-gate path is specified but untested.
All three inputs are reachable: a one-turn over-cap fix leaves `Spent` 1 of the default 2, so an attempt is left, and a two-turn one leaves none.

### [LOW] Per-kind rows in cards 35 and 36 repeat the predicate's own test

**Location:** `_lyx/plan/35-driver-hands-back-landing-halts.md:26`
**Issue:** `decideDrive` in card 35 sees the stop only through its `HandBack` flag, and `Check` in card 36 sets `LandingHalt` only from `landingshed.HandsBack`, so rows that differ only in which kind of landing halt the reason text names exercise no distinct code.
`TestHandsBack` (card 34) already pins the producer-and-prefix predicate for a landing-stop reason at each row, the undecodable-record and lock re-check halts and the reroute-cap reasons.
The acceptance criterion lists the halt kinds, but one marked row per consumer plus the predicate's own table covers each kind once, where the per-kind rows cover the same decision up to five times.
Card 36's `TestCheck_RefusesLandingHalts` is itself a new function justified: no existing test drives `Check` with a blocked status, since `TestEscalationEligible` drives only the awaiting path, so the finding is the per-kind rows and not the function.

### [LOW] Card 40 states no bound on the lock it now keeps

**Location:** `_lyx/plan/40-idle-release-keeps-finalize.md:27`
**Issue:** The idle release exists to free the record of a stalled loop.
Card 40 makes both paths skip a running Finalize and a live verify marker, and `KeepsLock` keeps the lock on a read error, so the card downgrades a guard.
The Intent says what the check keeps but not what ends the hold, which the attack-surface question requires of a card that downgrades a guard.
The bounds exist in the plan (owner death and takeover in card 27, the session timeout in card 11, marker liveness, the operator's release verb) and need one sentence naming them.

### [LOW] Card 7 leaves open whether merge-in mode refuses an empty aligned tip

**Location:** `_lyx/plan/07-fixer-session-table.md:30`
**Issue:** The rule that gives the stencil a non-empty start tip on a run's first Merge-In is consistent across the three cards the focus names: card 7 defines `AlignedTip` as the merge record's aligned tip or the entry's `AlignFrom`, card 11's `run` fills it from the `tips` argument, and card 26 passes the aligned tip or `AlignFrom` and pins that the first Merge-In's spawned table carries the fork point.
What card 7 does not say is whether the table enforces it.
It lists `AlignedTip` among the table's `Optional` values, and seatengine renders an Optional blank value as nothing, so a caller that passed two empty tips would give the `merge-in` stencil an empty start tip with no error.
Making the optionality finalize-mode only costs one condition in `FixerTable` and lets the seat engine's existing blank-value check fail loud on that programmer error.

## Rejected

- Card 26's `Halt` statement that every halt other than a spent cap and an over-cap change leaves `Halt` empty is imprecise for a halt raised before `align` runs, such as a conflict Stuck or a clean-tree halt on the resume, which leaves the earlier `over-cap` note in place; this is harmless, since no fixer turn runs in between and the next resume that reaches `align` still reads the operator's review as intended, so it is no finding.
- The over-cap halt resumed after the parent moved merges the new parent commits before the baseline is written afresh, so they never count towards the cap, and the aligned tip then advances over commits no fixer read; D4 settles this for the spent-cap resume in the same words, so the plan follows the decision.
- The retry pass after a merge-in-required error at Finalize keeps the entry's `Baseline`, so parent commits the retry merges count towards the cap; D4 and D10 accept this over-covering rather than letting a fixer change escape.
- `lens-review.md` in the plan directory is the lens review `internal/loomengine/plan.go` requires the lead to record there, not a stray file.
- Card 52's PATTERN.md replacements net -13 bytes against the file as it stands, as the decision record's headroom entries require.
