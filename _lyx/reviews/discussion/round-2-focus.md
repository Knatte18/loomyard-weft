---
round: 2
focus:
  - "decision-record.md, the Decision that maps a landing commit on the parent branch to a run: check whether the mapping source exists for every loom landing, including one whose parent run-records checkpoint commit was a no-op because the records were already clean, and whether the Decision states what a landing matching no source becomes and what bounds the fallback."
  - "decision-record.md, the Decision and the Constraint that make the report's readers tolerant of unknown fields: check whether the tolerance is confined to the report's own read path, or reaches readers that live runs and gates call with strict decoding (known-fields or disallow-unknown-fields decodes of agent verdicts and run state), and whether the Decision states what bounds it."
  - "decision-record.md, the Decisions that read records from git objects without a lock: check whether any Decision directs a path-based or locking reader to be called on archive-tag blobs, and whether the parse entry point is stated consistently across the record-reading Decisions."
  - "decision-record.md, the rule that bounds the batten-record read for a run: check what it selects for a slug whose run directory was removed by cleanup, and for a slug with no later fresh seed."
  - "decision-record.md, the halt-note Decision: check that it states where halt notes are read from, what a run with no halt-note source reports, and what a halt note with no matching batten change counts as."
  - "decision-record.md, Acceptance criteria: check whether each classification rule that carries a requirement has a criterion a test can fail independently of the all-sources fixture, filing a gap once as a method finding."
---

Entries restate their sites and make no severity claim.
