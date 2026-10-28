```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1, F2 and F3 and check the fix covers every member.

- **F1 (set: history-only outcomes that record no working spawn):**
  - The `Outcome` constants in `internal/shedengine/producer.go` are Done, Stuck, Awaiting, Yield, `OutcomeGoto`, `OutcomeReroute` and `OutcomeSkipped`.
  - `internal/loomengine/coherence.go:76` says Goto, Reroute and Skipped are the history-only outcomes, and Yield is not a valid history outcome there.
  - `latestDarnOutcome` now skips all three, so the fix covers the whole set, not only the two the review named.
  - The other readers of `OutcomeReroute` (`battenshed/classify.go`, `loomcli/checklist.go`, `reportengine/rounds.go`) answer different questions and consume no Darn-Write prior-work outcome.
  - A search of `loomshed` and `darnengine` found no stale doc comment about reroute handling.
- **F2 (set: error branches of `latestDarnOutcome`):** the status read error and the non-not-exist stat error are the two non-trivial error returns. The status read error is not part of this finding. The stat error now has a row.
- **F3 (set: callers of `latestDarnOutcome`):** the only non-test caller is the `darnDeps` closure at `darnwiring.go:158`, and the new `TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile` covers it.

No findings.

Unchecked: none.

Tool calls:
1. Grepped `Outcome` constant declarations and `latestDarnOutcome`/`OutcomeReroute` call sites. My context lacked the full outcome set and the other readers of reroute.
2. Grepped `loomshed` and `darnengine` docs for stale reroute wording. My context did not cover those packages.
