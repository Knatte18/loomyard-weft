---
round: 1
focus:
  - "internal/landingshed/finalize.go, finalize_pr.go, prreview.go, parentalign.go: the merge of the four-value step return into one ending type. Settle whether every former return path (including error wraps and early exits) keeps the same outcome, with no behaviour change."
  - "internal/landingshed/finalize.go, fixer.go, fixerattempts.go, stuck.go: the lock-freed, requeue and rebase layer around the fixer. Settle whether the diff designs it away or leaves it, and whether the spec's invariants in docs/research/rewrite/shrink-estimate-2026-11-18/landingshed-spec.md still hold."
  - "internal/landingshed/deps.go, verifygate.go, describe.go, fixreview.go, fixerreport.go: removal of single-caller seams and adapters, including the parentPuller interface. Settle whether any removed seam was reached by a production caller, and whether merged fixer report parsers and stop-result wrappers keep each parser's distinct error handling."
  - "internal/landingshed/prreview_wait_test.go and internal/shuttleparity/seats.go: the test edit and the caller edit outside landingshed. Settle whether the test change is only a signature change and whether the seats.go change is licensed by the Scope."
  - "Commit messages and _lym/landing/summary.md, when present: settle whether the line counts before and after per spot, the comparison with the study's estimate and the spots left unsimplified with their reasons are all reported, and whether the diff bears them out."
---

Seeded from the decision record's Goal and the diff stat of the review range.
