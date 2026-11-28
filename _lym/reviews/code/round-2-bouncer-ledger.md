---
round: 2
ledger:
    - key: refusal-post-key-name
      rounds:
        - 1
      status: resolved
    - key: merge-danger-understated
      rounds:
        - 1
      status: resolved
    - key: refusal-episode-end-untested
      rounds:
        - 1
      status: resolved
    - key: unattended-reads-as-on-untested
      rounds:
        - 1
      status: resolved
    - key: transient-failure-ends-episode
      rounds:
        - 1
      status: resolved
    - key: stamp-lock-held-across-post
      rounds:
        - 1
      status: resolved
    - key: halt-notices-drop-refusal
      rounds:
        - 1
        - 2
      status: open
      class: decision
      severity: LOW
      fix: design-wrong
    - key: refusal-episode-test-tier-unnamed
      rounds:
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
---
The round-2 review reports every round-1 finding resolved except the one on halt notices that drop a refused autonomy value.
That one stays open for the parent: no added entry in the decision record settles it.

The new key is a test from a round-1 fix commit (`ce6dee700`) that does not name its tier.
