---
round: 1
focus:
  - "internal/alignment Session (card 6): does Begin reuse an attempt by key and never spawn twice, and is a closed attempt never polled again, as the alignment-idempotence-key decision states?"
  - "Parent-review gate and PR-Align producer (cards 7, 10): does every failure path fall back to today's delivery or PR-Gate path, with no verdict applied from a dropped or stale attempt?"
  - "Shared approval write (card 9): do the existing approve and reject verbs and the alignment apply path all go through RecordApproval and RecordRejection, with no behavior change for the verbs?"
  - "Alignment record paths (cards 4, 5): do records land under shedrun.AlignmentDir as the alignment-record-layout decision states, and does the status commit stage the alignment directory?"
  - "loom.yaml alignment keys (card 1): are the layer classes and the timeout-below-wait check implemented as the card specifies, and does ResolveAlignment honor alignment_pr_approve: false?"
  - "Test economy: do the new tests in each package duplicate an existing test, and does cmd/lyx/testdata/test-budget.yaml need the edit CLAUDE.md requires?"
  - "Docs (CLAUDE.md): do doc.go, docs/overview.md and PATTERN.md land in the same commits as the new module, verb and invariants?"
---

Seed focus drawn from the plan's Shared Decisions and Card Index.
Entries name where to look and which question to settle; none pre-states a verdict.
