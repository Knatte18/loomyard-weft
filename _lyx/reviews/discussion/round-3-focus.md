---
round: 3
focus:
  - "decision-record.md D3 (state changes): for every state and cause, including `unknown` with no signal, `unknown` from an idle-notice contradiction, `asking` from an interactive turn end with an output file missing, and `idle-done` reached after output files appear, does the text name exactly one `since` stamp and its source? Settle whether any state or cause is left without one."
  - "decision-record.md D1 (a line without `lyx_at` reads with the time the reader first saw it) against D3 (the signal-derived history is the same for every reader): `lyx shuttle state`, batten and the orch watcher read files only and persist nothing, so settle which time a reader without memory uses and whether D3's same-history claim is limited to lines that carry `lyx_at`."
  - "decision-record.md D3 and D6, and support-log.md Rejected alternatives (persisted state file): for `dead` with cause `process-gone` and `unknown` from liveness or an unreadable file, `since` is a call-time fact. Settle whether the text states how `since` behaves across repeated calls, and whether the stated reason for rejecting a persisted file covers those states."
  - "decision-record.md D1 (the Stop and ask hook rewrite and its Bound): the wait loop classifies turns by reading the events file that the Stop hook appends to, and the Decision requires every hook command to exit 0 whether or not the append succeeds. Settle what happens to the Stop and ask lines when stamping fails or the append fails, what that lets through for the wait loop in a task whose premise is that shadow mode changes no behavior, and whether the Decision states a bound."
  - "decision-record.md D1 (Bound) and D3 (`busy` and `interrupt`): the reducer compares a transcript timestamp with the newest turn start's `lyx_at`. Settle whether the state, not only `since`, depends on the agent-writable stamp, whether the Decision says so, and whether a tie or skew rule exists."
---

Each entry names a place and a question; none presupposes the answer.
The remaining areas of the pair are unchanged and need no extra attention beyond the normal rubric.
