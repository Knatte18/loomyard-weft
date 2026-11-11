```yaml
lens: bound-breaker
outcome: reported
```

The claims I attacked, held or broken:

- D1: `discussion_assist_*` keys are category `none`, in the template and open to `run_settings` (config.go:657-659). Held.
- D2: a goto and a resume restart nothing (bouncer.go:238, 367-369). Held.
- D2: the clear archives the run directory, re-seeds and carries the generation record (archive.go `archiveRunDir` calls `CarryRecords`). Held.
- D2: a new generation archives the run directory and seeds round 1 (amendment.go:42-59). Held.
- D2: a goto appends a history entry naming its target, and a run of identical calls folds into one entry with `LastAt` (goto.go:140, run.go:725-739). Held. A folded entry is only a budget-exempt Stuck, so a Discussion-Write done entry never folds; the claim is harmless.
- D2: "a goto, a resume and a restarted row keep" the round-1 focus file. Held: only `seedCall` and `ensureFocus(1)` write it (bouncer.go:1621-1651, 1000-1012), and a judge or replay writes round n+1's focus, never round 1's.
- D2: a seed is told apart by its write time. Partly open: the record never says at what precision the ask record stores the time. See F3.
- D2: a `not-sent` ask is retried next round "so a raised chair timeout takes effect at the next round". Broken; see F2.
- D2 bound: the parent's answer never gates the round, and approval stays the Bouncer's. Held.
- D3: `NotifyParent` falls back to the prime's orch (notify.go:41-49), returns posted false with a nil error when no strand is recorded (:193-195), reports no receiver (:47-50), and signs as a non-orch strand claim (:188-218). All held.
- D3: a restarted row posts again on `pending-post` and stays silent on any later state. Held under D5's forward-only lock.
- D4 guard: "admits the run's parent orch … and the operator's own unnamed shell". Held as a description of `decisionCallerRefusal` (decision.go:666-701). The empty-strand case admits any process without a strand name, not only the operator's shell, but D4 already states that the guard stops only a mistaken call. Held as stated.
- D4: an answer is accepted on `pending-post` or `open`, and refused at or past the deadline under the lock. Held.
- D4: the refusal for a superseded `--ask` names the way forward `lym loom decision add`. Broken; see F4.
- D4: a done report is copied "as `keepAssistReports` does" once the table ends, and archived on a failed exit. Partly broken; see F5.
- D5: the cap formula matches `CapAssistTimeout` (review.go:64-84) less a 5-minute margin. Held. Measuring the deadline from the ask, before the chair launches, only makes it earlier. A zero `review_timeout_min` falls back to `run_timeout_min`, and shuttle's 0 there means a deadline equal to the start time, not unlimited (shuttleengine/config.go:29), so "no room" stays "no room". Held.
- D5: state moves only forward under one lock, so an answer and a timeout never both land. Held, provided the row's `pending-post`→`open` write is conditional. An answer can land between the post and that write, and "moves only forward" covers it.
- D5: an ask the chair left without running the await is closed at table end. Broken for an ask answered `done` that the chair never read; see F1.
- D7: remapping `decision` to `design` keeps a parent finding from escalating. Held: the escalation cause `decision` keys on the verdict class (burlerengine/verdict.go:55, bouncerdecision.go:198).
- D6: a fork shares the orch's strand name, and its reach equals the orch's. Held and stated.

Findings:

F1. MEDIUM, design, D5 and D7 (decision-record.md:159-160, 193-197). The claim that "an ask the chair left without running the await is closed" fails when the ask is answered `done`. Counterexample:
- The chair writes its review without running `lym loom parent-review await`.
- The parent's fork answers `--findings-file` before the table ends, so the ask goes `done`.
- At table end Go leaves a `done` ask alone, and D4 copies the report beside the review as if it had been consolidated.
- The review carries no `origin: parent` finding. If the Bouncer converges, the parent's findings are lost silently.

"Keeps every parent finding" rests on stencil text alone, and the record states no bound for it. Evidence: burlerengine/review.go:267 copies after the chair ends whatever the chair read, and decision-record.md:159 closes only `pending-post` or `open` asks. Fix: the review row checks a `done` report with findings against the review's `origin: parent` findings and fails the review gate when any is missing. Otherwise D7 must state the instruction-only bound and mark its criterion `Review:`.

F2. LOW, design, D2 (decision-record.md:74) with D5 (:149-151). "A raised chair timeout takes effect at the next round" does not hold if the room is resolved where the assistant's bound is. The loom driver resolves `loomCfg` and the review bounds once, when it builds the recipe (wiring.go:612, 669, `segmentAssistBounds`). A `review_timeout_min` raised mid-perch then changes nothing until the driver restarts, and every round of the same seed writes `not-sent` again with a Warn. The record never says where the room or the per-round key read is resolved. Fix: state that the room and the ask key are read per round from fresh config, or reword the claim to "after the driver restarts".

F3. LOW, design, D2 (decision-record.md:62, 68). The seed is "told apart by its write time", but the record never says at what precision the ask record stores that time. The Bouncer's own time stamps are RFC 3339 at whole seconds (shedengine/run.go:627-629). If the ask stores the seed's modification time at one precision and compares it with the file's nanosecond modification time, the two never match. Every round then reads "seed differs" and asks again, and a restarted row cannot find its own ask (:95). Fix: state that the stored stamp is the file's exact modification time, compared exactly, or identify the seed by a content hash.

F4. LOW, consistency, D4 (decision-record.md:128-130). The refusal for an `--ask` that is not the latest number always points at `lym loom decision add`. When the latest ask is a newer `open` ask of the same run, the way forward is to answer that ask's number. Counterexample: the fork answers ask 3 after round 4 has asked ask 4, and the refusal sends it to record a parent decision instead of answering ask 4. That breaks PATTERN-refusal-way-forward's rule that a refusal names the right way forward. Fix: a superseded ask whose newer ask is still answerable names `--ask <latest>` as the way forward.

F5. LOW, consistency, D4 (decision-record.md:131-132, criterion :241). "As `keepAssistReports` does" carries over its exit set. `concludeReview` copies only on a nil run error or `ErrNotStarted`. On `ErrSeatNotStopped` or a shuttle run error it returns before the copy (burlerengine/review.go:252-262, 267), so on those failed exits no copy exists to archive. The criterion "a failed review exit archives that copy" holds only for a failed exit that passed line 267, such as a chair that died or a failed review gate. Fix: name the exits that copy, or make the copy independent of `concludeReview`'s early returns.

Tool calls:
1. grep of `run_timeout_min` in shuttleengine and of webster fork refusals: to check the D5 fallback's zero meaning, which the manifest lacked.
2. grep of `EscalationDecision`, class `decision` and `DiscussionMaxBounces`: to check D7's no-escalation claim and D2's zero-bounce claim one hop past the record.
3. grep of `segmentAssistBounds` callers: to find where the room inputs are resolved, for D2's "next round" claim.
4. Read of wiring.go:600-675: the build site of `loomCfg` and the review bounds, which the manifest did not cover.
