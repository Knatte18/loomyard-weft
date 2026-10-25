```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal, branch or test row a card adds, judge whether its state can occur and whether the guarded site comes back on an unguarded path.

**Per guard, reachable or not**

- Card 3, unknown-gate-name refusal (four legal values, `parent-review` now unknown): reachable through a recipe gate entry, and the replacement test reaches it.
- Card 3, in-flight row in `TestSequence_FullRunBlocksAtMergeIn` (an open round on disk is ignored): reachable. It models a directory left on disk, not a shuttle wait already holding the gate. I could not confirm whether that wait persists anything keyed by entry name, so this is not a finding.
- Card 4, `reviewWaitingFor`'s chain ending in a step that reports no wait: reachable from `lyx loom status` (`checklist.go:135`, `arm.go:318`).
- Card 6, retired-key drop in `MigrateConfig` and `TestPrepareRunSettings`' read-time migration row: reachable.
- Card 12, `routeDecisions` returning handled false:
  - switch off: reachable (row 14);
  - review does not parse: reachable (row 15);
  - escalation record of another cause: reachable only for a round an older binary escalated (row 16);
  - decision file of another cause: reachable only from an older binary or a switch flip (row 17);
  - settled accept: reachable, but on the replay entry it does not take "today's path" (finding 2).
- Card 12, "None" branch (escalate, or the same Awaiting on a re-call): reachable (rows 1 and 4).
- Card 12, `settledGeneration` false for an unsettled `decision` round: reachable, since the escalation over CONVERGED writes no settle record, so `reenterUnchanged` does not pre-empt it (`settlerecord.go:166-189`).
- Card 12, pending continue:
  - dropped-key refusal leads to a re-judge, and at the budget to Shed's budget block: reachable (row 11);
  - continue at or past the hard cap leads to a re-escalation with cause `decision`: reachable only through a lowered `HardCap` or an amendment cap, as the card says (row 12).
- Card 12, pending accept:
  - refusal below the budget leads to a re-judge: reachable (row 7);
  - refusal at the budget below the hard cap leads to retire and re-escalate: reachable (row 8);
  - refusal at the budget at or past the hard cap leads to the `DecisionHaltPrefix` Halt: reachable (row 10).
- Card 13, `sendDecisionFindings` posting nothing under `DecisionsEscalated`: reachable. `TestBouncerEntry_BlockingEscalatesAndColorSegment`'s judge-spawn harvest does reach the routing (`entries_bouncer_test.go:1266`). The site recurs unguarded (finding 3).
- Card 14, Eligibility refusing a pending `decision` escalation: reachable through `EscalationEligible` and `Check`.
- Card 14, `DecisionHalt` refusal in `Check` and batten's `HandBack`: reachable from `lyx loom arbiter` and from batten's re-step. The guarded site recurs unguarded at the Go driver's tackler (finding 1).
- Card 15, rename with both keys present (new value kept): reachable.
- Card 16:
  - `HandsBack` matching `retiredMergeInName`: reachable, since batten reads a child's status file before any rename;
  - old attempt-record fallback and its decode refusal naming the old path: reachable;
  - `fixerReportInput` fallback: reachable;
  - `recipeRow` mapping through the retired-row tables: reachable;
  - `renameGateKeys` dropping the old key beside a new one: reachable.

**Findings**

1. BLOCKING, design, card 14 (`14-decision-cause-readers.md`, "Blocked stops").
   - **Summary:** the decision halt card 12 raises is withheld only from `arbiterstop.Check` and batten's `HandBack`. The loom Go driver still hands a blocked Discussion-Bouncer stop to the tackler, an agent that can resume it with a goto to its own row and spend a judge spawn each time. That contradicts the card's own bound, "only the run's parent lifts" it.
   - **Evidence:**
     - The loom spec's `HandBack` is `landingshed.HandsBack` alone (`internal/loomcli/arm.go:308`, `loomCLI.specFor`), and so is the Go driver's `HandsBack` (`internal/loomcli/drivewiring.go:320`, `loomCLI.goDriverDepsFor`).
     - `stopFromStatus` computes `HandBack` from that predicate (`internal/sheddriver/driverules.go:110`).
     - `decideDrive` sends any other blocked stop to `driveActionTackler` (`driverules.go`, the `state == blocked` case). The arbiter is reached only after a tackler resume has returned (`facts.ResumeReturned`).
     - `TacklerCall.conclude` and `resumePrecondition` admit a resume whose goto is the current row (`internal/sheddriver/tackler.go:371`, `:395`; `shedengine/goto.go:149`).
   - **Fix:** card 14 also edits `internal/loomcli#loomCLI.specFor` and `internal/loomcli#loomCLI.goDriverDepsFor`, so their hand-back predicate is true for a blocked Discussion-Bouncer status whose error opens with `shedadapters.DecisionHaltPrefix`. Ideally `Check`, batten's wire and these two sites share one predicate. A test reaches it through `decideDrive` or `stopFromStatus`.

2. MEDIUM, design, card 12 ("Routing").
   - **Summary:** the settled-accept handled-false branch is meant to take today's path, but on the replay entry it no longer does. Card 12 makes `standsAsRecorded` false for every round with cause `decision`, settled or not. A settled `decision` accept reached through `settledEntry`'s `entryReplay` (a settle record that is not live) then goes to `settleUnlessRefused`, where `routeDecisions` returns handled false. That reaches `judgmentRefusal`. Over a CONVERGED verdict whose review holds a BLOCKING finding the accept passed over (row 9's BLOCKING `decision` finding, or a BLOCKING `design` finding the parent accepted after a re-judge), it refuses and re-judges a settled round. Today the decided round stands as recorded.
   - **Evidence:**
     - `bouncer.go` `Call`: `if b.standsAsRecorded(n) { return b.settle(...) }; return b.settleUnlessRefused(...)`.
     - `settledEntry`'s `entryReplay` (bouncer.go, "one that is not live replays through the round-by-round path").
     - `judgmentRefusal`'s CONVERGED BLOCKING checks (bouncer.go, after `standsAsRecorded`).
     - `liveSettleRecord`'s conditions for a record that is not live (`settlerecord.go:166-189`).
   - **Fix:** `standsAsRecorded` returns false for a `decision` round only while its decision file is absent or unsettled. Alternatively, `routeDecisions`' settled-accept case calls `settle` directly. Add a row to `TestBouncer_DecisionEscalation` for a replay over a settle record that is not live.

3. LOW, design, cards 12 and 13 (`routeDecisions` handled-false paths against `decisions_escalated`).
   - **Summary:** the fix row's silence assumes the Bouncer escalates every `decision` finding. On the handled-false paths a `decision` finding reaches no one: no fixer, no fix-row notice, and no `decision` brief. This happens when the round already carries a `circling`, `budget` or `hard-cap` record, or a decision file of another cause (the upgrade cases). Row 16 escalates with cause `circling`, whose brief does not list `decision` findings.
   - **Evidence:**
     - Card 13 Intent: `sendDecisionFindings` posts nothing when `DecisionsEscalated` is set.
     - Card 12 "Routing": handled false for an escalation record of another cause, and for a decision file of another cause.
     - `burlerfix.go:67`: `DecisionsRouted` withholds the findings from the fixer.
   - **Fix:** on those branches, either list the round's `decision` findings in the brief, or state the accepted bound in card 13's Intent.

**Tool calls**

1. Grepped `internal/shuttleengine/gate.go` for gate state keyed by entry name, to check whether card 3's in-flight claim holds for a run mid-gate. The manifest did not cover shuttle's gate persistence.
2. Read `layoutBouncerRound1Report` and the `TestBouncerEntry_BlockingEscalatesAndColorSegment` rows in `internal/shedrecipe/entries_bouncer_test.go`, to check that card 13's new row reaches the routing.
3. Read `liveSettleRecord` in `internal/shedadapters/settlerecord.go`, to learn when `reenterUnchanged` and `entryReplay` fire for a `decision` round.
4. Read `CheckGotoTarget` in `internal/shedengine/goto.go` and grepped `drivewiring.go` for `GotoGuard`, to confirm a tackler goto to the blocked row's own producer is admitted.
