---
round: 1
ledger:
  - key: refusal-post-key-name
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: merge-danger-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: refusal-episode-end-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unattended-reads-as-on-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: transient-failure-ends-episode
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: stamp-lock-held-across-post
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: halt-notices-drop-refusal
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round, so every key is new.

The open findings cluster in two places.
The dobby's handling of a refused autonomy value covers the post's wording, when an episode ends, the lock around the post and the halt notices that drop the refusal.
Test coverage misses the episode end and the readers that ask only whether autonomy is on.
The change description's Merge danger call is the one finding outside the code.

The review rejected the role-text and glossary gaps for `unattended` as outside the entry's scope.
