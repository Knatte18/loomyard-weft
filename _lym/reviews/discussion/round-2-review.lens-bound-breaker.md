```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims and bounds and tried to build a counterexample against each one.

**Claims attacked**

1. D2: a settled decision file of cause `decision` from round N-1 is shown to round N as the parent's answer. **Broken.**
2. D2/D3: the `Added after Discussion` entries past round N-1's snapshot count are the answer to each held `decision`-class finding. **Broken** (it is the same defect as 1, so it is filed with it).
3. D3: the round-check entry in `internal/burlerengine/verdict.go`, given only the run directory and round number, and `lym burler validate-review` from the review file's run directory, can run every round check. **Broken.**
4. D3: `ParseCurrentReview` keeps its shape checks alone, so `acceptReview` and `acceptedReview` accept a review that passed the round checks on the cap. **Broken.**
5. D4: in an incremental round, no reviewer or fork can claim the `origin: parent` exemption. **Broken.**
6. D6: "The decision escalation keeps its place ahead of the judge." **Broken.**
7. D1: the fix check passes on its cap and so never archives the round or spends a bounce. **Held.** The check joins the engine's own table as `ReviewGateEntry` does, not `opts.Gate`, so the skip path's `checkGates` never runs it.
8. D1: `spawned` is a Go-only key, refused when a fixer writes it. **Held.**
9. D5: a review re-run in the same round keeps the first record, so a dead fixer's edits never become the round's start. **Held.** The failed-gate archive list does not include the record.
10. D6: the soft-cap auto-accept comes first. **Held** (`settleUndecided` at `bouncer.go:1355`).
11. D6: a replay settles as the harvest did, including a `review_stalemate_rounds` edit between them. **Held.** `escalateWith` keeps the recorded cause (`bouncer.go:1514-1516`).
12. D6: a stalemate escalation never counts against the budget. **Held** for the escalation itself. A later `continue` counts as it does for `circling`.
13. D6: `continue` keeps its hard-cap refusal. **Held** (`internal/loomcli/circling.go:116-124`).
14. D4: with no changes file, the BLOCKING rule holds alone. **Held** for new findings. The carried-finding move rule is unstated there (see finding 6).
15. Open risks: an in-flight round with no snapshot or frontmatter runs as today. **Held.**

**Findings**

1. **BLOCKING, design**, decision-record.md D2 and D3: the parent's answer to a `decision`-class finding can never reach round N.
   - A settled decision file never precedes a round N. `settled` is legal only on an accept (`internal/shedadapters/circling.go:232`), and an admitted accept passes the round through `acceptRound` and ends the segment (`internal/shedadapters/bouncerdecision.go:173-174`, `internal/shedadapters/bouncer.go:1313-1330`). A refused accept is retired by archiving (`bouncerdecision.go:178`, `bouncer.go:1284-1290`).
   - The other channel cannot tell which finding was answered. `Added after Discussion` entries belong to the whole record, not to a finding (`bouncerdecision.go:354-373`). D2 still renders every new entry beside every held finding, and D3's round-check entry forces each such finding to `resolved` and stops carrying it.
   - Counterexample: round N-1 holds decision findings F2 and F3. The parent runs `lym loom decision add` for F2 only. Round N's block shows that entry beside F3 too, the gate refuses `open` on F3, and F3 drops out unanswered.
   - Fix: drop the decision-file source. Render the new entries once in the block. Let the reviewer mark each held `decision` finding `resolved`, with an `answer` naming the entry, or `open`. The entry then refuses only a `resolved` whose `answer` names no new entry.
2. **MEDIUM, design**, decision-record.md D3, the round-check entry and `lym burler validate-review`: the inputs the record names cannot settle the checks it lists.
   - The checks need state in four places outside the review file's run directory and round number: the acceptance record (`roundComplete`, `internal/shedadapters/burler.go:36-47`), the generation record's `Amendment` mark (`internal/shedadapters/amendment.go:82-88`), the decision files, and the decision record's added entries (`bouncerdecision.go:354`).
   - Those formats are parsed in `shedadapters` and `discussionparser`, and `burlerengine` cannot import `shedadapters`, since `shedadapters` imports it (`internal/shedadapters/burler.go:14`).
   - `validate-review` resolves no hub or worktree (`internal/burlercli/cli.go:66-68`), so it cannot find the decision record at all.
   - Counterexample: `validate-review` on an incremental Discussion-Review whose decision finding was answered cannot tell that it was answered, so it disagrees with the gate.
   - Fix: the review producer builds a round context: whether the round is incremental, the earlier findings with their answered flags, the touched files, and the ask's ids. It hands that to the entry, and `validate-review` rebuilds it through the same builder.
3. **MEDIUM, consistency**, decision-record.md D3, "`ParseCurrentReview` keeps the shape checks alone": those shape checks refuse D3's own key.
   - `checkReviewKeys` refuses any top-level key but `verdict` and `findings` (`internal/burlerengine/verdict.go:184-200`).
   - Counterexample: an incremental review with a `prior` list fails the review gate on every attempt. It passes only on the cap, and then `acceptReview`'s strict parse fails the round (`internal/burlerengine/handoff.go:55-58`).
   - Fix: say that the shape check admits `prior`, and refuses it only where the round checks do.
4. **MEDIUM, design**, decision-record.md D4, the `origin: parent` exemption: the claim that no reviewer or fork can claim it is false.
   - The reviewer is the one who writes `origin: parent` and `parent_ids` (`contracts/stencils/burler/burler-review-format.md:51-52`, `contracts/stencils/burler/burler-parent-consolidation.md:16`), and `Origin` is never checked (`verdict.go:73-78`).
   - Counterexample: in a round with an answered ask holding P1, the reviewer tags a new NIT on an untouched file with `origin: parent, parent_ids: [P1]`, and it passes.
   - Fix: state the real bound, which is only a round with an answered ask. Or have Go check that a parent-origin finding's id set is not already covered by another finding.
5. **LOW, consistency**, decision-record.md D6: "The decision escalation keeps its place ahead of the judge" is false.
   - `routeDecisions` runs inside `settleUnlessRefused` with the judge's verdict and ledger (`bouncer.go:1037-1038`), and every caller runs after a judgment (`bouncer.go:433, 457, 508`).
   - Fix: say "ahead of the refusal check and settle".
6. **LOW, design**, decision-record.md D3 and D4, the carried finding's location move: the move rule is unstated when no changes file exists.
   - D4 says only that the BLOCKING rule holds alone when the cache is gone. Whether a carried finding may move its location path then (always refused, or always allowed) is unspecified.
   - Fix: state it. Recommended: no move without a changes file.

**Tool calls**

1. `repairReportBeforeGate` and the external callers of the review check: to see whether a fix check would sit in `opts.Gate`, where `checkGates` would run it on the skip path.
2. The callers of `ReviewGateEntry`: to see how the review check joins its gate, which settled claim 7 as held.
3. The `origin: parent` and `parent_ids` lines in the burler stencils: to find who writes parent-origin findings.
