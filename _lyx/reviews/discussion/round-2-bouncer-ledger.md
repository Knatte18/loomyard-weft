---
round: 2
ledger:
  - key: pause-expires-held-request
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: shared-window-pane-moved-in-session
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: marker-removals-escape-lock
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: withdraw-refusal-keyed-on-phase
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: reconcile-erases-pane-id-d2-reads
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: grace-spawns-ps-per-read
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-not-git-repo-skip-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pause-notice-without-live-watcher
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d811-holder-state-unverified
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: tmux-fact-duplicated-with-review-narrative
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-double-spawn-bound-misnamed
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: removal-edges-unbounded-condition3
    rounds: [1]
    status: resolved
  - key: d2-guard-already-exists
    rounds: [1]
    status: resolved
  - key: missing-launch-time-premise
    rounds: [1]
    status: resolved
  - key: tmux-empty-answer-unverified
    rounds: [1]
    status: resolved
  - key: unnamed-false-live-cases
    rounds: [1]
    status: resolved
  - key: landing-conflict-stop-skips-shell-only
    rounds: [1]
    status: resolved
  - key: d6-no-reclaim-inside-grace
    rounds: [1]
    status: resolved
  - key: d7-skip-misses-post-resolve-teardown
    rounds: [1]
    status: resolved
  - key: acceptance-omits-windows-skip
    rounds: [1]
    status: resolved
  - key: child-shell-risk-lacks-evidence
    rounds: [1]
    status: resolved
  - key: departure-restates-rejected-alternative
    rounds: [1]
    status: resolved
---

Round 2 re-raised four round-1 keys: the pause-held request now fails one step later, inside its requested cycle's handoff timeout (F1, BLOCKING); the shared-window target does not exist in reed (F2); the marker's writer sits outside the state lock (F4); the withdraw departure is unrecorded (F5).
Round 1's F3 fix introduced the shared-window text, so F2 reverses it rather than contradicting a settled finding.
Round 1's other keys do not recur and are resolved.
Several round-2 keys are follow-ons of round 1's fixes: the launch grace's ps spawn, the D7 skip's bound, the duplicated tmux fact.
New design-level key: reconcile clears the pane id D2 looks up.
F7 is judged a new key rather than a recurrence of `d7-skip-misses-post-resolve-teardown`, since it concerns the skip's bound, not a post-resolve teardown.
