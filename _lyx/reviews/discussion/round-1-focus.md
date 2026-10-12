---
round: 1
focus:
  - "decision-record.md D2 and D3: settle whether the sweep's lock-taking on a prefixed entry, and the lock-less-past-grace removal rule, can delete files of a live process, for example a run directory whose owner created it just before the grace boundary, or a child's directory beside a parent when the parent's TMPDIR root is detected."
  - "decision-record.md D6 against D4 and Open risks: the scan admits an allowlist entry for a site that cannot move, an escape hatch; settle whether the Decision states what the allowlist can let through and what bounds it, and whether it agrees with the support log's Question ledger Q3 (`empty allowlist`)."
  - "decision-record.md D9 and D11: for each destructive segment, settle whether the stated bound covers every deletion path, namely the tracked-but-deleted candidate, the aged orphan rule, the `.scratch` link removal and the board-citation check, and whether a removal can reach a running or still-referenced task's state."
  - "decision-record.md D7 and D8: settle whether the dry-run/`--apply` equivalence, the in-process segment calls and the continue-on-failure rule leave a segment able to remove something its own verb would refuse, and whether D8's premise about fabric and `pairteardown` holds."
  - "decision-record.md D1, D4 and Constraints: settle whether the trust-boundary rule for the root and the Windows removal ordering are decided consistently across D1, D4 and Constraints, and whether every requirement the plan writer needs from support-log.md's Interview (as opposed to its narrative) is present in decision-record.md."
---

Focus targets the attack-surface question (new verbs, an allowlist escape hatch, destructive sweeps) and the consistency of the stated bounds.
None of the entries touches the rubric's `Do not flag` items: no entry asks for rejected alternatives in the record, a plan-writer notes section or call-site enumeration.
