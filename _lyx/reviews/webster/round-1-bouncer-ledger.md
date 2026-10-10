---
round: 1
ledger:
  - key: untested-models-yaml-refusal-discussion-advisors
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: doc-comments-name-discussionwrite-as-only-seam-reader
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two findings against the nine-commit range.
F1 (BLOCKING, scope): the `LoadConfig` refusal for an unreadable `models.yaml` with a non-empty `discussion_advisors` has a spec row but no test reaches it.
F2 (LOW, consistency): three doc comments in `internal/shedrecipe` name the DiscussionWrite entry as the only user of seams the DiscussionSeats entry now also uses.
