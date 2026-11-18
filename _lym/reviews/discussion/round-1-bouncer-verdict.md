---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING findings F1, F2 and F3 (F3 and F2 cite declared PATTERN entries) and gating design findings at MEDIUM F4, F5, F6, F7 and F9; no relabels."
---

Round 1 is the first round, so no fix has been reviewed and nothing converges.
F1, F2 and F3 are BLOCKING design findings.
F2 cites `PATTERN-test-speed` and F3 cites `PATTERN-child-processes`, whose declared change in this task admits the Hardener sandbox reaper beside `lym tender reap` as an `EndProcess` user, bounded to processes under the sandbox root that no record names; the fix for F3 must stay inside that text.
F4, F5, F6, F7 and F9 are `design` at MEDIUM and gate on their own.
F8 (`consistency`, MEDIUM) and F10 (`decision`, MEDIUM) are not gating by class.
F10 goes to the run's parent and never to the fixer.
F11, F12 and F14 (`design`, LOW) and F13 (`consistency`, LOW) do not gate.
No finding was relabeled and no focus departures were present.
