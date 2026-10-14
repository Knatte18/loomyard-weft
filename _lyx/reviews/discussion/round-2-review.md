---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:153 (D11)
    summary: D11 migrates each document on its own discussion_producer, but whether a list was live depended on the producer resolved across layers, so a worktree override silently gains or loses advisors
    fix: In D11 add two rules and an acceptance row for each. First, a document whose discussion_producer is single or twill gets discussion_advisors set to the empty list, written explicitly even when the document set no list, so a lower layer's list stays off where that document turned it off. Second, a document with discussion_producer set to seats and no discussion_advisors of its own gets a report line saying its advisors came from a lower layer whose migration may have emptied them, and asking the operator to set its named list. Extend the MigrateConfig acceptance criterion with a worktree file of twill over a seats default with advisors, which loads with no advisors, and a worktree file of seats with no list, which reports the line.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:106 (D7)
    summary: D7 lets a local override or run_settings start any number of advisor seats, a reach the retired guard key denied, with no reason given and the guard-key alternative neither taken nor rejected
    fix: Make discussion_advisors a guard key, as D8 makes discussion_lenses, so only the hub-wide file or a worktree's own config sets it. In D7 replace the sentences from With discussion_producer gone through No maximum count is set with one stating that the key is a guard key and so starts advisors only where discussion_producer could. In D8 change the closing claim so the guard bounds advisors and lens forks alike. In D11 drop the local-layer and run_settings cases of the string-entry refusal, since ValidateLocal refuses the guard key there, and adjust the matching acceptance criterion. Add the category-none alternative to support-log.md's Rejected alternatives. If per-task advisors are wanted instead, keep category none but state in D7 why, and add the guard-key alternative to Rejected alternatives.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:63 (D3)
    summary: D3 shows an attempt's verdict without naming its source; verdict.yaml lacks the verb's own hand-on for a died, timed-out, missing, unparseable or precondition cause, so such an attempt would read no verdict
    fix: In D3 state that the verdict comes from the attempt's envelope (loomengine.ReadArbiterEnvelope at ArbiterEnvelopePath), shown as its verdict word with the cause in parentheses when Cause is set; with no envelope the line reads running before Deadline and no verdict after it; an envelope that cannot be read shows verdict unreadable and never breaks the frame. Add an acceptance criterion for an attempt whose envelope carries a hand-on with a died cause.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:67 (D3)
    summary: An unreadable attempt goes to the end of the segment's block, yet its stop key, and so its segment, is unknown; a run with several review segments, or a stop outside any, has no such block to name
    fix: In D3 place the unreadable-attempt line at the end of the frame, after the history-only rows and before the state and wait lines of D1. Change the matching clause of the arbiter acceptance criterion at decision-record.md:206 from at the end of the segment's block to at the end of the frame, before the state line.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:52 (D2)
    summary: D2 rounds review-row entries by the review file their output names, but a goto history entry names its target row with an empty output, so a goto into a segment row has no round
    fix: In D2 add that a goto history entry naming a row of the segment stands as the last line of the round in progress, or before round 1 when none has opened, and that the review visit after it opens the next round by its output as the existing goto sentence says. Extend the goto acceptance criterion at decision-record.md:205 so the goto line renders at the end of round k and round k+1 opens with the review visit.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:52 (D2)
    summary: D2 says a hand-on entry opens no round, but a restart over an accepted review whose own entry was never persisted makes the hand-on the first entry naming its round, which must open it
    fix: In D2 replace joins the round already open and opens none with joins the round its output names, opening it only when no earlier entry has, and add that clause to the hand-on acceptance criterion at decision-record.md:203.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:70 (D3) and decision-record.md:204
    summary: D3's rationale and an acceptance criterion name an arbiter continue verdict, but the verdict words are resume and hand-on
    fix: Replace continue with resume in D3's rationale at decision-record.md:70 and in the acceptance criterion at decision-record.md:204.
---

### [MEDIUM] D11's per-document migration misreads advisors across layers

**Location:** `_lyx/discussion/decision-record.md:153` (D11)

**Issue:** D11 keys every `discussion_advisors` rewrite on the `discussion_producer` of the same document, and `MigrateConfig` is a pure rewrite of one document (`internal/configsync/configsync.go` calls it per file in `reconcileSparseFile` and both default reconcilers).
Whether a list was live, though, depended on the producer the layers resolved to, and a worktree's own file may set `discussion_producer`, a guard key.
Two cases follow.
A hub default of `seats` with advisors under a worktree file of `twill`: the default's list migrates to named entries, the worktree file gains `discussion_lenses` and nothing else, and the advisors start in that worktree where they were dormant, the very harm D11's rationale names.
A hub default of `twill` with advisors, today's hub, under a worktree file of `seats` with no list: the default's list empties, and the worktree's advisors vanish without a report line.
The focused cases otherwise hold: a string list in a local file, a sparse worktree file or `run_settings` without `discussion_producer` passes `MigrateConfig` untouched, and `ValidateLocal` checks only the mapping shape, so it reaches load, where D11 refuses it.

### [MEDIUM] D7 widens advisor reach without a reason

**Location:** `_lyx/discussion/decision-record.md:106` (D7)

**Issue:** Today advisors start only when `discussion_producer` is `seats`, a guard key no local override or `run_settings` can set.
D7 keeps `discussion_advisors` category none, so any layer, a board task's `run_settings` included, can now start advisor sessions, with no maximum count.
D7 states this reach and bounds each entry, so the attack-surface rule is met on its face, but nothing in the Goal or the rationale asks for per-task advisors, and making the key a guard key, as D8 does for the lens key, keeps today's reach at no cost to the bench, whose pair configs are worktree files.
D8's "bounds no more than the key it replaces" holds for the lens forks only, and D8 says so; for advisors the bound is wider than the replaced key's.

### [MEDIUM] D3 names no source for an attempt's verdict

**Location:** `_lyx/discussion/decision-record.md:63` (D3)

**Issue:** An attempt directory holds the agent's `verdict.yaml` and the verb's `envelope.json`.
`internal/loomcli/arbiter.go` concludes an attempt with a hand-on for the causes `timeout`, `died`, `missing`, `unparseable` and `precondition`, which leaves no valid `verdict.yaml`, and `loomengine.ReadArbiterVerdict` refuses an invalid one outright.
D3 says "its verdict, or, with none, `running` ... `no verdict`", so a reader of `verdict.yaml` renders a verb-concluded hand-on as `no verdict`, hiding why the run stopped.
The attempt reader itself now holds: D3 reads each directory with `ReadArbiterAttempt`, so one bad `attempt.json` no longer fails the listing.

### [MEDIUM] An unreadable attempt has no segment block to sit in

**Location:** `_lyx/discussion/decision-record.md:67` (D3)

**Issue:** D3 places the unreadable-attempt line "at the end of the segment's block, since its number and stop key are unknown".
The unknown stop key is what places an attempt, so its segment is unknown as well.
A loom run has several review segments, and an arbiter may arbitrate a stop at a D1 row, so "the segment's block" picks no block.
The acceptance criterion at line 206 repeats the phrase.

### [MEDIUM] A goto entry has no round

**Location:** `_lyx/discussion/decision-record.md:52` (D2)

**Issue:** `shedengine` records a goto as a history entry whose `Producer` is the target row, with outcome `goto` and no output (`internal/shedengine/goto.go`).
A goto into the review row thus writes a review-row entry that names no review file, and D2's only placement rule for review-row entries is the review file its output names.
The sentence at line 54 places the review visit after the goto, not the goto entry itself; gotos into the Bouncer or fix row are equally unplaced.

### [LOW] A hand-on can be the first entry of its round

**Location:** `_lyx/discussion/decision-record.md:52` (D2)

**Issue:** The output rule maps a hand-on to the right round: an unjudged hand-on names round `highest`'s review, and an accepted-review hand-on names round `highest+1`'s (`BurlerReviewProducer.Call` in `internal/shedadapters/burlerreview.go`), and `shedadapters.ParseRoundReviewName` is the exported parse D2's owning-package clause can use.
The added claim "opens none" is false when the reviewer's accepted Stuck was never persisted, a crash between acceptance and Shed's persist, since the restart's hand-on is then the first entry naming round `highest+1`.

### [LOW] `continue` is not an arbiter verdict

**Location:** `_lyx/discussion/decision-record.md:70` (D3) and `decision-record.md:204`

**Issue:** `loomengine` declares the verdict words `resume` and `hand-on`; a resume with no goto continues where the run stands.
The backticked `continue` reads as a verdict value a test would match.

## Rejected

- Placement of a Bouncer entry that routes nowhere, D2 against D3: not a defect.
  D2 places such an entry last in the round in progress and any arbiter line after it, D3 places the attempt after the entry its stop key names, which is that same entry, and the next Bouncer entry that bounces opens round n+1 under D2, so the two agree.
