# Round 1 fixer-report — Discussion decision record

## Changes

All changes are in `_lym/discussion/decision-record.md` unless named otherwise.

- F1: fixed as proposed. D9 now checks whether step 3 is done first, and otherwise runs D6, the approval-hash match and D8's re-check before step 1. D8's re-check runs again inside step 3. Steps 1 and 2 write only into the run's `_lym/raddle/`, each output carrying the approved proposal's hash. A step is skipped only on a hash match. A D8 return deletes the failed attempt's step 1 and 2 outputs. The D9 retry criterion covers a re-check failure at step 3 followed by a changed, re-approved proposal.
- F2: fixed as proposed. The Codex Open risks line is deleted. A Constraints fact cites the rollout's token fields. D12 gives Codex a `UsageReader` over the last `total_token_usage`, with cache-write zero and input equal to `input_tokens` less `cached_input_tokens`. A D12 criterion for a Codex session is added. The fact names `reasoning_output_tokens` from Codex's token reports, not from the rollout fixture, since only the bench report shows it.
- F3: fixed as proposed, with one difference. D12's writer records each session's dollar cost, priced per message through the engine's `MessagePricer` via `shuttleengines`, and unknown when no price exists. D13 sums those costs, and the `prices` key is gone. The D13 criterion is updated. The `PATTERN-no-hard-coded-choice` bullet is deleted. A new `PATTERN-shuttle-provider-seam` bullet would duplicate the existing one, so its pricer rule is merged into that bullet instead. A fact cites Claude's pricer and `tools/tokencount`. A new Open risk says that Codex has no pricer, so its runs enter the dollar fit in hours only. Support log: Q14 is revised, and the price table is added to Rejected alternatives.
- F4: fixed as proposed. Step 2 writes the part records into the run tree only. Step 3 stores each record and sets its discussed flag in the write that creates the entry, through a new `boardengine` write. A Constraints fact cites `StoreDecisionRecord`'s refusal.
- F5: fixed as proposed. D7 records the mode at Raddle-Write's first spawn, and later sessions and Raddle-File read the recorded mode. Raddle-File runs D6 in both modes. The rule is in D7's Bound, D6 and D9, and has its own D7 criterion.
- F6: fixed as proposed. A D8 return clears the recorded approval, and the D8 criterion covers it.
- F7: fixed as proposed. In auto mode the first gate pass stores the cold proposal, and the last gate-passing proposal is the approved one. The D7 auto criterion covers it.
- F8: fixed as proposed. The waiting notice goes out on the first pass of each Raddle-Write session, and the cold proposal is still stored once per run. The D7 criterion is updated, and support log Q16 is revised.
- F9: fixed as proposed, taking the halt-notice option. The run's entry stays done at step 3, since F16's marker rests on it. Raddle-File halts with a reason naming the step, and the halt notice carries it. This is in D9's Bound and the D9 criterion. Support log: Q19, plus a Rejected alternatives line for the final-write option.
- F10: fixed as proposed, taking the discussed-flag option. D2 keys the promote rule on the discussed flag with a stored record, and promotion restores a dropped `sketch` label. The Bound and the D2 criterion are updated. Support log: Q17.
- F11: fixed as proposed, taking the fallback. The board keeps no status for a removed entry, and `internal/boardengine/dependency.go` already reads a missing slug as pruned done. D8's Bound states that a removed entry's dependency is dropped as if landed. Support log: Q18.
- F12: fixed as proposed. D6's first check requires every PATTERN change to be carried by exactly one part. The D5 and D6 criterion is updated.
- F13: fixed as proposed. Seed records whether the run started from a stored record, D13 keys its cells on that too, and both criteria name the flag.
- F14: fixed as proposed. In D13, a run with no Finalize size record enters the mean only, and such a cell reports no fit. The D13 criterion covers it.
- F15: fixed as proposed. D8's second bullet compares by `revised` later than the baseline, and the D8 criterion is updated.
- F16: fixed as proposed. D9 marks step 3 done when the run's entry is done and every filed slug exists, and checks that before the re-check. The D9 criterion covers it.
- F17: fixed as proposed. Constraints facts cite the run lock's `scopeChanged`, `SetRunStatus` and `MakeSketch` skipping a done entry, and `Store.Prune` keeping a `notify_on_land` entry. Step 3 clears `notify_on_land` in the same write, and step 5's notice stands in for it.
- F18: fixed as proposed. D10 writes the store inside step 3's one board write and commit. The D10 criterion covers another machine reading it.
- F19: fixed as proposed, with one difference. The `PATTERN-run-usage-record` bullet says a cost report reads token counts and session costs only from those files. It also names costs, since F3 puts them in the same files.
- F20: fixed as proposed. The stencil's "too few runs" rule moves into D4 and into the D4 Review criterion. The Open risks line keeps only its risk.

## Self-check

The changes below came from the self-check and were made after the forks returned.

- fix-consistency, LOW: `PATTERN-shed-recipe-registry` bullet disagreed with the recorded mode. Fixed as proposed.
- fix-consistency, LOW: D8's return count did not cover D9's D6 returns. Fixed as proposed: D8 counts D6 and approval-hash returns, and so does the D8 criterion.
- fix-consistency, LOW: the Codex fact read as the end state. Fixed as proposed: "in this tree".
- fix-consistency, LOW: the D13 heading was stale. Fixed as proposed.
- fix-claims, LOW: the halt notice does not name a step today. Fixed as proposed: Raddle-File halts with a reason naming the step, the notice carries that reason, and D9 states this as new work. The D9 criterion is updated.
- fix-proposal, MEDIUM: D6's filing-time `depends_on` check failed on an entry D8 counts as landed. Fixed with my own change, in D6 itself rather than only the filing-time run: the check admits a `board_checked` entry gone from the board. A new D6 and D8 criterion covers it.
- fix-proposal, LOW: Open risks line 1 was stale after the flag split. Fixed as proposed.
- fix-way-forward, LOW: a failed hash match had no outcome. Fixed as proposed: it returns to Raddle-Write naming the mismatch and counts among the returns. The D9 criterion is updated.
- fix-way-forward, LOW: a D6 return named moved entries. Fixed as proposed: the return names the failing checks.
- fix-way-forward, LOW: D2's discussed-flag key catches a discussed note that never became a Sketch. Fixed by stating in D2's Bound that this is intended, since its record would seed D3 the same way.

## Disputed

None.

## Deferred

None.
