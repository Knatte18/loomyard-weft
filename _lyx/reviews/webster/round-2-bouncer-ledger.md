---
round: 2
ledger:
  - key: comment-fixed-column-wrap
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: websterengine-doc-stale-model-assertion
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: plan-review-rubric-stale-check-tally
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: publish-rejected-push-zero-remote-commits
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: expired-shell-warning-passthrough
    rounds: [2]
    status: open
    class: design
    severity: NIT
---

Round 1 raised four findings; round 2 re-raised each of them in a new form and added one NIT.

- `comment-fixed-column-wrap` (F1): the round-1 fix covered only part of the diff, and many comments and three new spec rows still break sentences mid-phrase or at non-clause commas.
- `websterengine-doc-stale-model-assertion` (F2): the package doc is clean now, but the `state.go` file header still names the removed asserted model.
- `plan-review-rubric-stale-check-tally` (F3): the rubric's range now stops at a row that is no longer last, and its every-check claim is false for `delete-target-gone`.
- `publish-rejected-push-zero-remote-commits` (F4): the zero-commit branch is covered, but an absent remote branch gets wording that says the branch exists, and no test row reaches it.
- `expired-shell-warning-passthrough` (F5, new): `expiredShellWarning` only forwards to `backgroundShellSentence`.

The review's class labels differ from round 1 for F2 and F4; the ledger copies the latest review's labels.
