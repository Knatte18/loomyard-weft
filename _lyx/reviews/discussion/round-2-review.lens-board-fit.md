```yaml
lens: board-fit
outcome: reported
```

I reviewed the board-fit lens: how the record fits the rest of the hub (board entries, peers, parent commits and decision notes).

**List**

- Entry `regression-baseline-verify` (T1, this task): I read its body earlier and held it against the record. Two `Added after Discussion` parent rulings (F9, F10) supersede parts of the body. The record follows F9 in D2 but not F10, and keeps an Open risks line that F9 settles; both are findings below.
- Decision notes titled `Decision for regression-baseline-verify:`: none in `lyx board list`, so there were none to read or miss.
- Peer `umbrella-anchor` (T3, Plan-Burler-Review, parent `main`), decision record read:
  - Overlap with no conflict. Its `lyx fabric anchor` refuses an unfinished run and a held gate slot, so a run with a live drift capture is refused there and is never re-anchored mid-capture.
  - Both records end a worktree's processes through `runproc.End`, the same mechanism. Neither changes that mechanism or a PATTERN entry differently.
- Parent commits listed by `lyx loom peers`: none.
- Note `csharp-sandbox-hub` (E28): overlap, no conflict. The record's Out section leaves the sandbox harness and the end-to-end proof to it.
- Note `run-verbs-shared` (E7): a text-only overlap and no finding. The record names `lyx loom goto` and `lyx loom resume` for darn halts, which E7 may later rename.
- Project `dotnet` (P2): this task fits its brief ("regression"). There is no conflict.
- Note `landing-escalate-arbiter` (E35): no overlap. It concerns landing-fixer escalation, not shed stop-chain hand-back of Drift halts.

**Findings**

1. MEDIUM · `design` · `decision-record.md` D2 lines 72–73, Acceptance criteria line 229, `support-log.md` Rejected alternatives line 317 and Q6 line 341.
   - Summary: D2 still says `Drift-Await` carries no entry gate and records Done with the flag off. That contradicts the parent's `Added after Discussion` ruling F10 (decision-record.md line 199–203), which gives `Drift-Await` `entry_gate: drift.enabled` and records it Skipped.
   - The first acceptance criterion and the Rejected alternative "`Drift-Await` behind `entry_gate: drift.enabled`" both keep the overruled design.
   - D2 should take the ruling, and D5 should then say what holds Webster back when the flag is turned off while a capture runs. Under the old design that was the stated reason for leaving `Drift-Await` ungated.
   - Evidence: decision-record.md lines 72–73, 199–203 and 229; support-log.md lines 317 and 341.

2. LOW · `consistency` · `decision-record.md` Open risks, lines 222–223.
   - Summary: the Open risks bullet asks the parent where `Drift-Verify` sits relative to `Parent-Align`. The `Added after Discussion` ruling F9 (lines 193–197) already answers that question, so the fix drops the line.
   - Evidence: decision-record.md lines 193–197 and 222–223.

Outside my lens: the peer record states that "`runproc.End` signals nothing" on Windows. That corroborates that D4's claim that the capture "ends with the run's processes" does not hold on Windows. The bound-breaker and existing-paths lenses own that.

**Tool calls**

1. `lyx board list`: required by the lens, to find touching entries and this task's decision notes.
2. `lyx loom peers`: required by the lens, to find peers and parent commits.
3. `lyx loom peers show umbrella-anchor --record decision`: the only close peer's record, which my context did not hold.
4. `grep` over the saved peer record for path, gate and runproc sentences: the output was too large to read whole.
5. `grep` for the peer's Windows runproc line: failed on a regex error and returned nothing.
6. `awk` over the saved peer record: extracted the peer's Windows `runproc.End` line and the `lyx fabric anchor` refusal list.
