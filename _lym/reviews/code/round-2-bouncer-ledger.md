---
round: 2
ledger:
  - key: resume-pushes-ungated-head
    rounds: [1]
    status: resolved
  - key: wake-pushes-before-foreign-head-check
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: board-run-settings-keep-retired-pr-modes
    rounds: [1]
    status: resolved
  - key: retired-pr-reviewer-role-breaks-agents-yaml
    rounds: [1]
    status: resolved
  - key: on-to-review-notice-misstates-meaning
    rounds: [1]
    status: resolved
  - key: foreign-head-order-before-endings-untested
    rounds: [1]
    status: resolved
  - key: merge-danger-blast-radius-understated
    rounds: [1]
    status: resolved
  - key: landing-for-decision-doc-stale
    rounds: [1]
    status: resolved
  - key: retired-key-comment-misstates-machine-layer
    rounds: [1]
    status: resolved
  - key: local-layer-pr-reviewer-load-untested
    rounds: [1]
    status: resolved
  - key: retired-loom-pr-local-and-run-settings-untested
    rounds: [1]
    status: resolved
  - key: pr-continue-strand-env-read-untested
    rounds: [1]
    status: resolved
  - key: in-flight-pr-reviewer-session-never-stopped
    rounds: [1]
    status: resolved
  - key: shellguard-publish-comment-names-removed-session
    rounds: [1]
    status: resolved
  - key: overview-lists-parent-review-ask
    rounds: [1]
    status: resolved
  - key: pattern-landing-summaries-omit-operator-ending
    rounds: [1]
    status: resolved
  - key: duplicate-beam-reinterpretation-test
    rounds: [1]
    status: resolved
  - key: told-worktrees-test-missing-keep-mark
    rounds: [1]
    status: resolved
  - key: template-defaults-test-missing-keep-mark
    rounds: [1]
    status: resolved
  - key: rubric-fixture-names-pr-findings
    rounds: [1]
    status: resolved
---

Round 2 resolved every round-1 key but one.

`wake-pushes-before-foreign-head-check` stays open.
The wake now checks for a foreign head before its gate push.
PR-Review's row entry (`openPullRequest`) still pushed before reading GitHub's head when the task head equalled the gated head,
so a hand commit on GitHub ended in a push halt whose way forward also fails.
The tests missed it because the wait world's push fake always succeeds.

The review also noted, without filing findings:

- A wording slip in `docs/overview.md` where "That ending" follows the hand-commit halt sentence.
- Local config layers on other machines are neither rewritten nor flagged, though the run halts naming the fix.
- A profiles.yaml rewrite case with a renamed value in a later entry is untested.
