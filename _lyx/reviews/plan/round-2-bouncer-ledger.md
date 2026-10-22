---
round: 2
ledger:
  - key: card10-static-prose-assertion
    rounds: [1]
    status: resolved
  - key: card7-warn-repair-unreachable-way-forward
    rounds: [1]
    status: resolved
  - key: card11-resume-noticed-clear-never-retried
    rounds: [1]
    status: resolved
  - key: reed-status-omits-elsewhere
    rounds: [1]
    status: resolved
  - key: card11-no-id-branch-unreached
    rounds: [1]
    status: resolved
  - key: strandremovable-doc-comment-stale
    rounds: [1]
    status: resolved
  - key: card9-restates-awakesince
    rounds: [1]
    status: resolved
  - key: card4-impossible-state-row
    rounds: [1]
    status: resolved
  - key: defensive-branches-for-impossible-wiring
    rounds: [1]
    status: resolved
  - key: card7-worktreetorndown-per-symbol-test
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card5-pane-lookup-warn-every-poll
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card7-skip-line-repeats-per-call
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card8-impactsummary-caller-count
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; the fresh round-2 review does not raise any of them again, so all nine are resolved.
Round 2 raised four new findings.
Three are BLOCKING design findings: card 7 plans a per-symbol test of `worktreeTornDown` on a false premise, card 5 logs a failed server-wide pane lookup at Warn on every polled call, and card 7 repeats its worktree skip line on every `Strands` call.
The card 5 and card 7 logging fixes add an episode guard, which is new design text needing a fresh review.
The fourth, LOW, is a wrong caller count in card 8's ImpactSummary.
