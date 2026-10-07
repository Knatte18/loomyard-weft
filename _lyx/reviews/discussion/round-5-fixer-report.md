# Round 5 fixer report

All three findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

## Changed

- F1: D5's wiring sentence now names the receiving set by constructing package: the hub-mode runners `loomcli`, `webstercli` and `burlercli` build notify, and the runners `shuttlecli` and `orchcli` build and every detached runner wire none.
  D5 says what each notifying runner carries (`loomcli`'s shared runner, the `recover-batch` recovery strand, the operator's `lyx burler`).
  The Auto-mode assumption repeats the same set in the same terms.
  The recovery strand `lyx webster recover-batch` spawns is now in the notifying set instead of holding silently.
- F2: the "batten-driven runs reach it through those verbs" clause is deleted from D5; batten builds no runner.
- F3: the Scope bullet and the D5 title now say "per live held turn end", and the Scope bullet states the Attach exception (at most the last turn end already in the file).
  An acceptance criterion now pins that an Attach over several held turn ends already in the file notifies exactly one.

## Deferred

None.
