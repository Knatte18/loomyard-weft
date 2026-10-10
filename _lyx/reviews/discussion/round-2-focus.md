---
round: 2
focus:
  - "Check the compaction session-start flow in decision-record.md (D11): the hook runs a recording command and `lyx orch resume-context` independently, and the watcher skips its typed pointer on a recorded session-start event. Settle whether the skip is conditioned on evidence the verb succeeded, and what the session receives when the verb fails."
  - "Check D11's choice between the resume prompt and the reload prompt against the whole orch state (`compacting`, `handoff_requested`, `resuming`, `PendingResume`), including a watcher tick that completes on the compaction boundary before the verb reads state. Settle whether the choice is fully determined and whether the handoff note can be lost."
  - "Check D5's premise that every lyx-booted session spawns the watchdog child. Settle whether sessions booted through reedengine by orch start, loom run/step or batten revival reach the discover signal, or whether the record states the delay for that group of paths."
  - "Check every new routing edge or wake source in the decision record (the discover signal in D5, the resize signal in D4) for a stated bound: who may write it, how often it can run a cycle, and that a wake finding no signal file runs no cycle."
  - "Check D7's readiness reading against the session-state fold: every state name and cause is covered by the ready or held lists, a held turn start has a stated release, and a session-start signal after a turn end leaves readiness unchanged. Check the acceptance criteria mirror the same lists."
---

Entries name where to look; none caps a severity or presupposes a verdict.
