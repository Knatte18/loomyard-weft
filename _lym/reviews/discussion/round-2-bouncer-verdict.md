---
verdict: CONTINUE
rationale: "Round 2 carries one BLOCKING design finding, F8 (D5 assumes the walk reaches every file before any load, but only start verbs run it, so in-flight runs halt on D4's refusal); BLOCKING always continues the loop, and F1-F7 are resolved."
---

F8 is BLOCKING and class `design`, so the decision rule gives `CONTINUE` whatever its key history.
Its key is first raised in round 2, so it maps to no key open in an earlier round.
No finding was relabeled.
The round 1 keys, mapped from F1-F7 by the review's `prior` list, are all resolved.
The review has no `## Focus departures` section.
