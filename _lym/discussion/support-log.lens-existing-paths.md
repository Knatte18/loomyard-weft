```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that reads, writes or resets the state that D1 to D6 introduce or change.

**Findings**

1. `MEDIUM` — D3, D4: the review gate passes on cap, so D4's bound is not a hard bound.
   - `burlerengine.ReviewGate` is registered with `Attempts: reviewGateAttempts` (3) and `PassOnCap: true` (`internal/burlerengine/verdict.go:221-255`).
   - So a reviewer that keeps raising a new non-BLOCKING finding, or one on an unchanged file, gets through after three re-prompts.
   - The record says the review check "refuses" these findings and states D4's bound as absolute ("cannot raise a non-BLOCKING finding"). It does not say whether the new checks live in the pass-on-cap gate or in `ParseCurrentReview`, which runs at acceptance in `internal/burlerengine/handoff.go` `acceptReview`, nor what happens at the cap.
   - The record must pick one of two and state it:
     - the incremental checks refuse at acceptance;
     - or, at the cap, Go drops or downgrades the offending findings.

2. `MEDIUM` — D1, D2, D6: Go-written fixer-reports have no frontmatter, and decision-only rounds count as unchanged.
   - `burlerengine.Fix` writes `skippedFixerReport` and `skippedRoutedFixerReport` itself when no fixer is spawned (`internal/burlerengine/fix.go:30-46, 141-203`). Neither carries the D1 frontmatter.
   - A round whose review held only decision findings that were routed to the parent therefore leaves a report that D2's prior-findings block cannot parse. D4's "complete round N-1" test must say whether it accepts that report.
   - The same round has an unchanged target because no fixer ran, so D6 counts it toward a stalemate. A `decision` escalation followed by a parent `continue` can then reach `stalemate` on the next quiet round, though nobody was asked to change anything.
   - The record must have Go write an empty `dispositions` frontmatter in both skipped reports, and say whether D6 counts a fixer-less round.

3. `MEDIUM` — D3, D4: `lym burler validate-review <file>` has no round context.
   - The verb runs `burlerengine.CheckReviewFile` on a path alone (`internal/burlercli/validate.go:24-57`).
   - D3's and D4's checks need round N-1's finding ids, its dispositions and the changes file.
   - As written, the verb either refuses every incremental review, because it reads a `prior` key as appearing in a full round, or it cannot run the checks.
   - The record must say what the verb checks in an incremental round: shape only, or the round's context found from the file's run directory.

4. `LOW` — D5, D6: a review half that runs again after a partial fix re-takes the snapshot.
   - The review producer moves earlier outputs out of the way when it starts again (`internal/shedadapters/archive.go` `archiveStaleOutputs`).
   - When a dead fixer is resumed through a re-run of round N's review, D5 takes the "round start" snapshot again, over the dead fixer's partial edits.
   - D6 then treats those edits as the before state. If the new fixer changes nothing, the round counts as unchanged, although the target changed during the round.
   - The record does not say whether the snapshot is written once per round or again on each re-run.

5. `LOW` — D6: `retireContinue` moves the decision file that D6's count restart reads.
   - `retireContinue` moves the escalation record, the parent notice and the decision file aside (`internal/shedadapters/bouncer.go:1276-1290`).
   - D6 restarts its count after "the latest round whose escalation the parent settled with `continue`", read from that decision file.
   - After a retire, that point is lost, and the count runs back across the earlier rounds.
   - The hard-cap re-escalation wins on that round anyway, so the effect is small. The record is still silent on it.

**Paths traced**
- D1:
  - `Fix`'s skipped-report writes: break (finding 2).
  - `fixTable`'s `repairReportBeforeGate`: survives; the new fix check can join `opts.Gate` the same way.
  - `bouncercarryover.go`, which cites the fixer-report by path only: survives.
  - `burler.go`'s `roundFixerReportPath` existence check, which decides round completeness: survives.
- D2: the `hydrationPaths` / `priorRoundsBlock` replacement survives. On a round whose prior report Go wrote, it breaks (finding 2).
- D3:
  - `ParseReview` readers (`bouncer.go`, `bouncerfacts.go`, `bouncerdecision.go`, `burlerfix.go`) survive if the strict parser admits `prior` and `carries`; older reviews without the keys still parse.
  - `parentCoverageGate` in `parentseat.go`: survives, since `origin: parent` is exempt.
  - The validate verb: breaks (finding 3).
- D4:
  - The review gate's pass-on-cap: breaks the bound (finding 1).
  - `archiveRunDir` on a new generation empties the run directory, so the next round is full: survives, and is consistent with D4's full-round rule.
- D5: `archiveStaleOutputs` on a review re-run is ambiguous (finding 4).
- D6:
  - `retireContinue` and the hard-cap re-escalation: minor break (finding 5).
  - The `decision` escalation order: wins on its own round, but the round still feeds a later count (finding 2).

**Tool calls**
1. Grepped for readers of the review file and the fixer-report: my context lacked the full consumer list.
2. Read `fix.go`'s skipped-report and gate wiring, and grepped the review readers in `handoff.go` and `parentseat.go`: my context lacked who writes fixer-reports besides the fixer.
3. Grepped `validate.go`, the round-file rename and remove paths, and the review gate's settings: my context lacked the validate verb, the archive paths and `PassOnCap`.
4. Read `archive.go` and grepped its callers: my context lacked how a re-run or a new generation treats round files.
