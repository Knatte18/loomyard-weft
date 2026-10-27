---
verdict: CONTINUE
rationale: "Round 2 carries six MEDIUM design findings; F1 (placeDobby refusal trace) reopens round 1's key pre-run-exits-silent-in-dobby-log and F2 (re-exec test not reaching recordStart or the pid writers) reopens reexec-test-by-pid-alone, both MEDIUM gating-class on keys open in an earlier round, so the loop has not converged."
---

The facts list no recurring key because ledgers carry no class, but the decision rule asks which round 2 findings map to keys open in round 1.

- F1 maps to `pre-run-exits-silent-in-dobby-log`. Round 1 asked for a trace of the silent exits before `Run`; round 2 shows the `placeDobby` refusals after a resolved worktree still leave none. Same key, MEDIUM, class `design`.
- F2 maps to `reexec-test-by-pid-alone`. Round 1 flagged the pid-only test at LOW; round 2 shows the ticks test still misses `recordStart` and the pid writers, at MEDIUM. Same key, class `design`.
- F7 maps to `adopt-check-before-migration` (LOW, not gating at this severity), F10 to `rotation-has-two-writers` (LOW).
- F3, F4, F5, F6, F8, F9, F11, F12 and F13 are keys first raised in round 2. F3 to F6 are MEDIUM design findings; they would go to Open risks on their own and do not block, but F1 and F2 do.

No finding is relabelled, and no focus departures section exists in the review.
No PATTERN entry is cited as violated; the review states none was found.
