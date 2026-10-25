```yaml
lens: coverage
outcome: reported
```

Coverage lens: each decision, acceptance criterion and new branch, mapped to the card that carries it and the planned test row that would fail if it broke. Below are the gaps only.

1. MEDIUM. The D2 in-flight criterion is argued, never tested. The criterion "A run waiting in a parent-review round passes Discussion-Write on its next gate evaluation under the new binary" has no row. Card 3 argues it holds because the row keeps its name and its gate list holds only `discussion`. `TestDiscussionWriteRow_GateListIsDiscussionAlone` pins only the recipe's gate list. Nothing evaluates the gate over a worktree whose parent-review round directory holds an open request with no verdict. Evidence: card 3 Intent, recipe bullet ("passes on its next gate evaluation … stays on disk unread"); decision-record.md:228.

2. MEDIUM. `lyx loom peers` misreads a peer still parked on `Merge-In`. The constraint says D2 and D11 change no row position. But `recipeRow` resolves a peer's position by its producer's name among the recipe rows, and answers 0 when the name matches no row. A peer whose status still says `Merge-In` has not been renamed yet: `renameRetiredRows` runs only in that run's own `armAt`, and never under a live driver on the old binary. Such a peer is listed at row 0 rather than Parent-Align's position. No card maps retired names through `LoomRetiredRows` in peers, and no row reaches this case. Evidence: internal/loomcli/peers.go:53, :80; card 16 "In-flight runs" bullet; decision-record.md:201.

3. MEDIUM. Card 12's `routeDecisions` has early returns no row reaches. Rows 1–14 always lay out a review that parses with a `decision` finding, or set the switch off (row 14). Two handled-false branches have no row:
   - the round's review does not parse;
   - the round carries an escalation record of another cause, such as a budget or circling record already on disk when a `decision` finding appears.

   The second branch decides whether an existing circling or hard-cap escalation hides a `decision` finding from the parent. Evidence: card 12 Routing paragraph.

4. MEDIUM. The way-forward constraint on the Awaiting Reason is unasserted. `PATTERN-refusal-way-forward` says the `decision` escalation's Reason names the way forward and a test reaches it. Row 1 asserts the brief lists the way forward, but no row asserts that `escalationReason` for cause `decision` names `lyx loom decision add &lt;slug&gt;`. Card 12 changes exactly that wording. Evidence: card 12 Brief paragraph (escalationReason), Tests row 1; decision-record.md:197.

5. LOW. Row 10 never runs the resume that re-judges after the hard-cap halt. The criterion "a resume re-judges the round" is not reached: row 10 stops at the Halt Stuck and the archived judge outputs, with no next Call showing a judge spawn under the recorded accept. Evidence: card 12 Tests row 10; decision-record.md:239.

6. LOW. The carry-over de-duplication branch has no row. `buildCarryOver` says "a ledger entry of the same key is not repeated", but row 6 does not lay out a `decision` finding whose id is also an open MEDIUM ledger key. Evidence: card 12 Carry-over paragraph, Tests row 6.

7. LOW. The snapshot's absent or empty decision record branch has no row. `writeDecisionSnapshot` writes an empty list for an absent or empty record, and every row has `withEscalationInputs`' record present. Evidence: card 12 Snapshot paragraph.

8. LOW. Row 5 does not assert the continue's budget exemption. A pending continue below the hard cap is `BudgetExempt` exactly when `budgetReached(round)`. Row 5 asserts Stuck and the focus file only, so neither the exempt nor the counted outcome is pinned. Evidence: card 12 Routing "pending continue" bullet, Tests row 5.

9. LOW. Card 13 has a branch with `DecisionsEscalated` set and `NotifyParent` nil. `NewBurlerFixProducer` sets `DecisionsRouted` when either is set, but the planned row always carries a notifier, so the `DecisionsEscalated`-only arm is unreached. Evidence: card 13 Fix rows bullet, Tests bullet.

10. LOW. Card 16's two fallbacks are tested only with the old file alone present:
    - `fixerReportInput` with both reports present, where the new one should win;
    - the old attempt record removed on both of its removal sites (the success path and the align path). The planned row names one removal.

    Evidence: card 16 "Code-Perch input" and "Attempt record" bullets.

11. LOW. Two read-time Warns are unasserted. The criteria "a profile or `run_settings` holding one loads with a Warn" (D3) and the D11 old-key migration at read time have rows that assert only that the key is dropped or renamed, not the Warn line. Evidence: card 6 Tests bullet (`TestPrepareRunSettings`); card 15 Intent; decision-record.md:229, :184.

12. LOW. D2's "No text names parent review" is incomplete. `internal/shuttleengine/gate.go:77` still says "Only the parent-review wiring sets it" (the `MayHold` field), and no card lists the file. After card 3 nothing sets `MayHold`, so the comment is wrong. `internal/creelengine/doc.go:202` names the parent-review gate in a past-stage design note, which may be left as history.

Tool calls past the context:
- `grep` of `internal/loomcli/peers.go` for position: my context lacked how peers computes a row position, which the D11 constraint depends on.
- `grep` of reportengine `wakes_test.go` and `fake_test.go` for superseded and verdict files: to confirm card 2's claim that existing tests decode the `verdict-&lt;token&gt;.yaml` superseded wake. They do, so no finding.
- `grep` for parent-review prose in `shuttleengine/gate.go`, `creelengine/doc.go`, `boardengine/doc.go`, `loomcli/resume.go`, `circling.go` and `shedrun/paths.go`: earlier counts showed hits in files no card lists.
