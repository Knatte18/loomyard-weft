```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fix carried out (F1, F3, F4, F5, F6) through the fix-proposal lens.

**Findings**

1. **MEDIUM** — `appendedDecisionRecord` in `internal/driftengine/judgesnapshot.go`, which now calls `CheckCarryOversNotLost(before, now)`. The F3 proposal makes a lym settle of an existing carry-over block a halt.
   - `WriteCarryOver` "replaces the segment's existing entry … When entry has no findings, it removes the segment's entry" (`internal/discussionparser/carryover.go:72-74`).
   - `writeCarryOver` in `internal/loomcli/wiring.go` commits that write and calls it from the review loop's settle.
   - Before the fix, `IdentityText` stripped every well-formed block, so a settle that replaced or removed a block was exempt. This was the "carry-over blocks … which lym settles" case the old comment named.
   - Now a settle that rewrites or removes a block while a judge session is open makes `CheckCarryOversNotLost` report "changed" or "dropped". The record is then named as the judge's edit and the run halts, the same false halt as issue 145.
   - Whether a settle can overlap a Drift-Verify judge session is not shown in the context I hold. The new `doc.go` bound states the opposite of the old comment: an edit or removal of an existing block is "still named as a change".

2. **MEDIUM** — the F4 refusal in `decisionVerb` (`internal/loomcli/decision.go`), which reads the scope record's `judge_session.Open`. The mark stays open while Drift-Verify is halted as a Stuck, because the doc says an open mark is "settled before anything else" at the next entry.
   - Case: a Stuck over a regression halts Drift-Verify with the mark open. The parent runs `lym loom decision add --amends-plan`, or a PATTERN-change form, to record the call that resolves it.
   - The verb refuses, saying to "re-run … once Drift-Verify has ended its judge session". The session can only end through `lym loom resume`, so the parent is forced to resume first.
   - The refusal's wording ("is open", "once Drift-Verify has ended") does not tell the parent a halt is not a running judge.

3. **LOW** — `driftJudgeSessionOpen` in `internal/loomcli/driftwiring.go` over `driftengine.ReadScope`.
   - `ReadScope` calls `recordLock`, which runs `os.MkdirAll` on the ephemeral drift directory (`internal/driftengine/records.go:261-263`). A read-only gate on a decision verb now creates a directory in every run, including runs with no drift.
   - A corrupt or unreadable `scope.json` returns an error from the new seam. The verb then refuses every `--amends-plan` or PATTERN-change call with `read the drift scope record: …`, an unrelated failure for a run where drift is off.

4. **LOW** — the F1 removal in `internal/driftengine/expectation.go`. Without the refusal, a range over a case whose lines are all keyed parses and seals.
   - Only the `seats.go` format sentence now prevents the issue-141 failure. A range written anyway shows up as an uncovered entry through `Uncovering`, which the gate does not turn into a re-prompt to the expect agent.
   - No test pins that sentence in the expect agent's rendered text.

F5 and F6: no findings. The F6 rows would fail with the exemption defects in place.

**Calls made**
1. `sed -n 60,135p internal/discussionparser/carryover.go; sed -n 555,590p internal/loomcli/wiring.go; grep -n "func recordLock" -A14 internal/driftengine/records.go`: to see what `WriteCarryOver` does to an existing block, who commits it, and whether `ReadScope` has side effects. My context did not hold the code.
