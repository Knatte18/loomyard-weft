---
round: 1
focus:
  - "D1/D2 reorder soundness: check that moving the portal and launcher teardown after the no-force dirty checks changes no verdict and no path other than the refusal one. Portal links and launcher dirs live under the hub, outside the warp worktree, so settle whether the worktree status probe, the force path, and a surfaceRefusal from removePortal/removeLaunchers (which still follows the archive under D1) all behave as the record claims."
  - "D3 vs the byte-identical-refusal constraint: nameStrandedPortalTeardown currently appends a reconcile remedy to the dirty refusals. Settle whether dropping that wrapper keeps the messages the livestate refusal matrix compares against destroy.go byte-identical, and whether D3, the Constraints bullet, and the acceptance criterion 'does not mention lyx fabric reconcile' agree with each other."
  - "Internal consistency between 'Notes for the plan writer' ('change is confined to remove.go, doc.go and the remove_*_integration_test.go files') and 'Open risks' (livestate matrix, mutation oracle and battencli lifecycle tests may need changes). Settle whether the note overstates confinement in a way that misleads Plan-Write, since Plan-Write reads only decision-record.md."
  - "D6 and the reuse acceptance criterion: after the reorder, settle which real sequence still leaves a same-tip archive tag on origin before a re-run (a teardown step failing after archiveWeftTip), and whether the record states that state precisely enough for the plan writer to seed the reuse test."
  - "Leanness of decision-record.md: D2 carries commit archaeology (9de0c46d6) and the history of the file header. Apply the completeness-before-leanness test: keep what the plan writer needs to rewrite the remove.go header and doc comment, and settle whether the remaining narrative belongs in support-log.md instead. Same question for the generic semantic-line-break Constraints bullet."
---

Seed rationale: the task is a narrow reorder inside `Topology.Remove`, so the risk sits in whether the reorder's claims hold against the current code and whether the record hands Plan-Write a consistent picture of scope.
Rejected alternatives and call-site enumeration are deliberately left out of focus, per the rubric's Do-not-flag list and the writer stencil's bounded-exploration rule.
