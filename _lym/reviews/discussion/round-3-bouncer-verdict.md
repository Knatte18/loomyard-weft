---
verdict: CIRCLING
rationale: "F26 is BLOCKING design on key d7-relaunch-prompt-startup, open in round 2 and still open (carries F22), so the rule gives CIRCLING; F27 and F28 are new BLOCKING design findings and would force CONTINUE alone. No relabels, no focus departures."
---

Round 3 carries three BLOCKING design findings.

- F26 maps to `d7-relaunch-prompt-startup`, listed open in round 2 and carried by the review as F22's startup-gate half.
  It is gating and still open, so the loop is circling on D7's relaunch startup.
- F27 (`d7-relaunch-record-after-reed`) and F28 (`d7-running-readers-skip-relaunching`) are first raised in round 3.
  They block convergence by being BLOCKING.
- F20, F21, F23, F24 and F25 are resolved, which closes `d7-relaunch-mark-unbounded`, `d7-relaunch-guid-orphans-callers`,
  `d6-copy-record-no-run-dir`, `orch-watcher-cannot-restart` and `d5-forged-input-bound`.

The review carries no `## Focus departures` section, so there is nothing to ratify or reject.
No finding's severity or class was relabeled.
