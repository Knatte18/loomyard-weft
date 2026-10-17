---
verdict: APPROVED
---

The design covers every Goal and Scope point of the `creel-notices` board entry and does no more.

- The orch watcher's and batten's notices and the watcher's injects go through creel (D5, D6), and every caller of the notice queue moves with them, tender's revive included (D7).
- The watcher's notice queue and its idle probe before typing go, and the watcher keeps only its cycle logic (D5).
  D9 keeps `Session.SessionIdle` for the hub-pause interrupt alone.
  That interrupt is a key, not a notice or an inject, and the record states it as a bounded departure, so it is not a scope deviation.
- A cycle phase advances on the orch's reply to the note request or on creel's outcome for a command inject (D4, D5).
  D5 also advances on `unconfirmed` for `/clear` and for steps gated by their own signal.
  Creel reports `unconfirmed` only after the line was typed, which is what the entry asks creel to confirm, and the record bounds the case.
- The note request is mail answered by post with `written <path>` or `defer` (D4).
- The reminder carries no content (D2), and the journal's `subject` and `lyx creel log`'s subjects stay, as the entry's idea says.
  The entry's two open points are settled in the question ledger (Q3, Q4).
- The notice log goes (D8), as the entry's orch recommendation says the task does.
- `PATTERN-durable-vs-ephemeral-state` gains the operator's sentence with "and the notice log" dropped.
  The same entry directs the task to remove the notice log, so the full sentence would name a file this commit deletes.
  Dropping it keeps the operator's rule unchanged, so I do not count it as a deviation.
  The record lists it as an open risk.

Nothing in the design belongs to another entry.
Scope Out leaves to `creel-agents` the notice wording, the stop-report de-duplication, agent-to-agent messaging, the parent-review gate, `lyx shuttle send` and `interrupt`, the reply discipline and reply chasing.
It leaves to `orch-split` the question of who receives an orch instruction inside a running task.
The note request's `defer` reply does not settle `creel-agents`' open question on whether `defer` counts as an answer, since this design adds no reply chasing.

## Overlaps

- Peer `shedrecipe-capabilities` (row 9, Plan-Burler-Review): both runs edit `internal/testkit/envkit/envkit.go`.
  This run removes the `NoticeLog.Now` entries, and the peer changes how the kit fills `Env.Segments` and `Env.Setting`.
  The edits are at different places in the file and change different behaviour, and neither decision record contradicts or repeats the other.
  Both may also move rows in `cmd/lyx/testdata/test-budget.yaml`, a shared bookkeeping file.
  Landing order: the peer is at row 9 and this run at row 3, so `shedrecipe-capabilities` lands first and this run takes its changes at Merge-In.
- `lyx loom peers` lists no landed parent commits.
