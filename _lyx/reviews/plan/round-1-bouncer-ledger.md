---
round: 1
ledger:
  - key: gone-worktree-quiet-exception-unbounded
    rounds: [1]
    status: resolved
  - key: absent-session-drops-run-lock-clause
    rounds: [1]
    status: resolved
  - key: batten-shutdown-refusal-bypasses-teardownrefusal
    rounds: [1]
    status: resolved
  - key: batten-bound-omits-inherited-deletions
    rounds: [1]
    status: resolved
  - key: endsessionbyname-no-bound
    rounds: [1]
    status: resolved
  - key: taskworktreepresent-stale-doc-comment
    rounds: [1]
    status: resolved
  - key: retire-during-wait-archive-untested
    rounds: [1]
    status: resolved
  - key: reed-status-envelope-lacks-retiring
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so there was no prior ledger to carry forward.
The reviewer raised eight findings, one BLOCKING: card 9 treated a gone task worktree as quiet without stating or bounding that exception to D2.
The rest were bound gaps (card 6's lock-free session end, card 11's inherited deletions), a dropped run-lock clause on an absent session, batten's Shutdown-path refusals bypassing `teardownRefusal`, a stale `taskWorktreePresent` doc comment, an untested retire-during-wait archive criterion, and `lyx reed status` not exposing the retiring flag.
The fixer addressed all eight, and the current plan carries each fix; all entries are resolved.
An independent pass over verify coverage, fidelity to the decision record, attack-surface bounds, granularity and `Custom` use found nothing new.
