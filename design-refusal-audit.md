Principle (operator-agreed): a false stop is as serious as a wrong landing; every refusal lyx issues must leave a documented way forward, and a guard that does not protect correctness warns and records instead of halting.

Evidence from 2026-09-30 (runs `operator-surface`, `batten-hub-ready`, `ide-hub-workspace`), each needing a hub code fix and deploy to continue:
- fork-audit false positives made a done batch unrecordable forever (flagged transcript never enters `seenForkTranscripts`) — `777d93af7`, `b848b8f9f`;
- run-entry and begin-batch validation refused a begun-but-unrecorded batch as create-already-exists — `b16efae35`, `4ab6f6dd6`;
- record-batch / begin-batch / recover-batch refuse each other in a ring after a parent merge (`webster-parent-merge-midrun`).

Scope: enumerate every refusal in `internal/websterengine`, `internal/webstercli`, `internal/shedengine`, `internal/loomcli` (preflight, coherence, fingerprint, audit, head-SHA, report-exists gates). For each: can a run reach it through no fault of its own? Is there a verb or state transition out of it? Is the halt needed for correctness, or can it warn?
Deliver: a table in the webster module doc (refusal, trigger, way forward), fixes for every refusal without a way forward, and tests reaching each from a realistic state.
Policy-class audit findings (fabric-reference, named spawn) on a batch whose report is OK and whose commits pass verify become recorded warnings, not halts.