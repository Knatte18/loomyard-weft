```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at this worktree's HEAD (fix-claims lens).

Claims checked:
1. D3: "as the `stalled` event is written once per change". **Does not hold exactly** (finding 1).
2. D3: "where the per-hub settle leaves a settled message alone". Holds: `settleDelivered` in `internal/creelengine/outbound.go:125-131` returns early when the message is not pending.
3. D3: "one hour from the post as for a message to another hub". Holds: `outboundDeadline = time.Hour`, "counted from its journaled post time" (`outbound.go:23-24`), and the deadline is `entry.Posted.Add(outboundDeadline)`.
4. D3: "A binary that does not know these events folds them to nothing, and the message stays an id only". Holds: the `default` case in `internal/creelengine/fold.go` logs at Debug and returns before appending the record to `p.events`.
5. Notes: "`internal/slackengine/keeper.go`'s notice for an archived orch channel names a rename Slack refuses". Holds: `keeper.go:196` reads "way forward: rename that channel in Slack, which frees the name". Whether Slack refuses the rename rests on the cited docs, and I did not re-fetch them.
6. Notes: "the per-hub `outboundWorker.run` … never sees machine-outbound mail". This holds by design. `pendingOutbound` and `outboundHubs` select entries by `ReceivingHub`, and D3 keeps machine-outbound entries off that path. It is a design statement, not a fact about current code.
7. D4: "Slack refuses a rename of an archived channel (… error `is_archived` …)". This is an external claim taken from the review. I did not check it against the code.

Findings:

- **MEDIUM**: D3, "report journals a `machine-held` event only when the reason differs … as the `stalled` event is written once per change".
  The precedent is described inexactly. The `stalled` event is also written again for an unchanged stall once the standing reading has gone stale.
  Evidence: `noteStall` in `internal/creelengine/deliver.go:736-745` appends a `stalled` record unless `standing && note.stall == stall && now.Sub(note.readAt) <= stallFreshness`. Its doc comment says it journals "when a non-empty stall differs from the standing one or the standing reading went stale" (`stallFreshness = 2 * probeBackoffCeiling`, `pool.go:100`).
  The change-only rule for `machine-held` is coherent by itself. Only the "as the `stalled` event" comparison misstates the code, and the review's wording carried the same imprecision.
  Fix: either drop the comparison, or say "as the `stalled` event is written on a change of reason".

Tool calls:
- One `sed`/`grep` of `internal/creelengine/fold.go:425-470` and `stallFreshness`. My context lacked the fold's handling of unknown events and the stall freshness constant.
