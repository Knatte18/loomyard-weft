```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every refusal and next step the fix introduced, and whether each can be carried out and leads somewhere useful.

Ways forward the fix introduced:
1. D1: a corrupt state file refuses, and the way forward is to repair it or restore its last committed version from the weft's history.
2. D2: `lym weaver reset` refuses while an attempt is in flight and its strand is live. The way forward is to re-step the row so it attaches, under `lym loom pause --after` if the run should stop once the row ends.
3. D7: a stopped batch whose own cards changed refuses, naming `lym weaver reset`.
4. D12: when the attach finds no run, the report is recorded directly from disk.
5. D11: every shared refusal names the active implementer's verbs in its way forward.

Findings:

- **MEDIUM** — D7, "A stopped batch whose own cards changed refuses, naming `lym weaver reset` as the way forward."
  This way forward can undo the edit that caused the refusal, so it may not make progress.
  Evidence: the fixed D2 says the reset "restores the plan directory in the weft to its content at the run's start".
  D7's own intro names the arbiter's fix and D7's rationale names the arbiter. Before this round, review F9 also named a Plan-Review reroute that rewrites pending cards.
  These are what change a stopped batch's cards. If the reset reverts them, a re-run meets the same plan and can stop on the same batch.
  Nothing stated lets the corrected cards survive the reset, so the loop can recur.

- **LOW** — D1, "restoring its last committed version from the weft's history".
  For Webster's state file this step may not exist mid-run.
  Evidence: Webster's state lives in the weft (`internal/websterengine/state.go:34` gives `_lym/webster`).
  The pre-fix support log (interactions LOW, fixed by D1's spec edit) says the spec commits the run directory "only at Done". So a Webster run in progress may have no committed state to restore.
  Repairing the file stays available, so the refusal still has one way forward, but the second one named is not always reachable.

No finding on items 2, 4 and 5: each can be reached from the state it names.

Tool calls:
- `grep` of `func Dir` in `internal/websterengine/*.go`, plus `ls` of `_lym/` and `.lym/`: to learn whether Webster's state lives in the weft, which my context did not show.
