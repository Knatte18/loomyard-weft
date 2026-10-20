---
verdict: APPROVED
---

Alignment at `parent-review` for `lenses-module`: the design matches the board entry's goal, with no scope deviation and no overlap with a peer run or a landed parent commit.

## Goal coverage

The board entry states its goal as five operator decisions under `Want (2026-10-19)`; it has no separate Done-when section.

- Lenses and fans leave burler for a `lenses` module with its file `lenses.yaml`, read by burler's reviewers and fixers and loom's twill writers: D1, with the readers in Scope.
  Darn's fan keys and the deregistration of `burler` follow from the move, since darn names burler fans today and `burler.yaml` holds no other key; neither is work of another entry.
- The template's `twill-discussion` becomes the seven named lenses: D5, in the entry's order.
- The hub `lenses.yaml` lists every lens and fan, reconcile carries a lens or fan the template adds and reports one the template drops: D2.
  D2 reports a dropped entry and keeps it rather than pruning it; the entry asks for a report and does not ask for removal, so keeping is within the goal, and the support log records the reason.
- A hub `burler.yaml` holding `lenses` or `fans` migrates into `lenses.yaml`: D4.
- The post-landing reconcile on the loomyard hub stays an orch action, out of the run's scope; D6 makes `hubreconcile.Ensure` perform the migration so that reconcile only confirms it.

## Other board entries

- `writer-reviewer-lens-align`: the seven-lens fan this task makes the template default is that entry's trial, and the operator decision in this entry's `Want` makes it the default.
  Scope's Out leaves that entry's measurement and its Plan, Webster and Merge halves open there, so nothing is duplicated or contradicted.
  D2 keeps the config-only trial route that entry relies on, now through `lenses.yaml`.
- `review-setup-audit`, `plan-writer-finding-classes` and `twill-writer-quality-audit` are audits that read the review setup; none plans a change to where lenses live, and D2 keeps their config-only route.
- `run-reentry` depends on `lenses-module` and changes burler's round and goto behaviour, not lens or fan config; no overlap.

## Peers and parent commits

`lyx loom peers` lists no parent commits and two peers, `lyxbin-timeout-test-xdr` and `watchdog-reap-test-xdr`, both at Merge-In.
Their changed paths are under `internal/testkit/lyxbin/` and `internal/reedcli/`, which this design does not touch, so neither overlaps and no landing order applies.
