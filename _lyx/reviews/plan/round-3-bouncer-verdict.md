---
verdict: CIRCLING
rationale: "F1 is BLOCKING (card 17 leaves the master-fork-running corpus replay failing), and F2 (design, MEDIUM) is the round-2 key gone-branch-remove-refusal-loops still open in a narrower form: a gone code branch beside another stop; F3 also reopens a round-2 key, and F4 and F5 are first raised in round 3; no finding relabeled, no focus departures"
---

Round 3 carries five findings: one BLOCKING, three MEDIUM, one LOW.

- F1 (BLOCKING, scope) maps to `card17-omits-broken-shell-wait-tests`, open since round 2.
  Card 17 now covers the tests round 2 named but misses the `claudeengine` corpus replay of `master-fork-running`.
  A BLOCKING finding alone forces `CONTINUE`.
- F2 (MEDIUM, design) maps to `gone-branch-remove-refusal-loops`, open since round 2.
  Round 2's fix sent a pure-gone stop through, but a gone code branch beside another stop still yields a refusal naming `lyx fabric remove`.
  The defect is the same, so a design-class finding on a key open in an earlier round makes this `CIRCLING`.
  The finding is narrower than round 2's, so the next fix is small and local.
- F3 (MEDIUM, scope) maps to `end-identity-check-rows-missing`: the group check for a descendant that leads its own group still has no `TestEnd` row.
  It is not gating-class.
- F4 (MEDIUM, design) is a new key, `paused-hub-resume-form-no-input`, first raised in round 3, so it does not block on its own.
- F5 (LOW, scope) is a new key, `busyerror-bare-teardown-literal`.

Round 2's `card17-stale-shell-wait-comments`, `live-parent-branch-refusal-unreachable`, `run-lock-holder-passes-unused-input` and `unpaused-run-lock-refusal-retry-never-succeeds` were not raised again and are resolved.
No finding is relabeled.
The review carries no `## Focus departures` section.
