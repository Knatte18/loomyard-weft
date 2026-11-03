---
verdict: CONTINUE
rationale: "F1 (MEDIUM design) reopens round-1 key detached-handover-untested, open in an earlier round, because spawnTeardownHold and the teardown child verb's RunE still have no test at any tier; F2, F3 and F4 are LOW and first raised in round 2, so they do not gate."
---

F1 maps to `detached-handover-untested`.
The round-1 fix moved the `childHold.finish` rows to tier 1, but the process boundary (`spawnTeardownHold`, its answer protocol, the lock-file record and the `RoleTeardown` registration) is still uncovered, so the key is reopened and gating.

F2 maps to the new key `teardown-signal-or-job-write-no-notice`, F3 to `unreadable-job-notice-omits-discard`, and F4 to `non-archive-failure-reported-as-archive`.
All three are LOW and first raised this round.
No finding is relabeled, and the review carries no `## Focus departures` section.

The other eight round-1 keys carry no finding in round 2 and are resolved.
