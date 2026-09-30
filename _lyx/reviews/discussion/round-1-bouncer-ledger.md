---
round: 1
ledger:
  - key: resume-once-marker-order-and-consumed-approval-signal
    rounds: [1]
    status: resolved
  - key: archive-tag-abort-point-force-and-offline-removal
    rounds: [1]
    status: resolved
  - key: done-wait-bounces-counted-against-running-budget
    rounds: [1]
    status: resolved
  - key: grace-elapsed-dirty-weft-state-and-recovery
    rounds: [1]
    status: resolved
  - key: records-pathspec-friction-archive-siblings
    rounds: [1]
    status: resolved
  - key: archive-scope-reaches-add-rollback
    rounds: [1]
    status: resolved
  - key: in-flight-split-friction-notes-consequence
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so there was no earlier ledger to carry forward.
The reviewer found seven problems in how decisions combine; none were gaps in the section contract.
The fixer resolved all seven in `decision-record.md`, adding matching support-log rejected alternatives and question-ledger entries.

Three were paths that no single decision stated:

- a single resume that fails to take, followed by an unbounded exempt wait;
- the archive-tag abort landing after `Remove` had already torn down part of the pair;
- a grace expiry that ends in a dirty-weft teardown whose only visible remedy was `--force`.

Of the rest, one was a budget-accounting error (done-wait bounces were counted against the running budget) and two were scope wording problems: the friction archive siblings, and `Add`'s rollback.
The last was an unrecorded consequence: split-run friction notes are lost at deploy.

The judge approved.
One non-blocking wording point is left and is not entered in the ledger: the phrase "after its refusals and before its first mutation" in the archive Decision.
In today's `Remove`, the dirty-tree refusals run after the portal and launcher teardown, so the phrase cannot place the tag.
The acceptance criterion pins the intended placement, before `removePortal`.
