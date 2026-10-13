---
verdict: CONTINUE
rationale: "Round 2 carries a BLOCKING finding (F1, class design): D5's attach-time window selection rides AttachArgv's layout chain, which returns the bare argv whenever the shell window holds fewer than two panes, so the common session never selects the relevant window; F2 and F3 are LOW consistency and gate nothing."
---

F1 is BLOCKING and gating, so the decision rule gives `CONTINUE` regardless of key history.
No finding was relabelled.

F2 (Warn on a recurring failure, against `PATTERN-log-levels`) and F3 (D5's newest-agent premise unqualified on psmux) are LOW consistency findings and do not gate.
F2 cites `PATTERN-log-levels`, which is an overview entry, so it stays at its stated severity and is not downgraded further.

The review carries no `## Focus departures` section, so there is nothing to ratify or reject.

None of round 1's keys recurs in round 2.
The review's focus verdicts settle the `internal/selfexec` leaf set, the explicit insert targets and the replace rule, and it raises none of the rest again, so the ledger marks all eight resolved.
