---
round: 1
focus:
  - "D2 (attack surface): record-batch now counts and re-audits the bracket's own transcripts. Settle what a bracket with a non-empty own list and zero new transcripts can now attribute or accept, for example a report written before the fork was re-driven, and whether the stated bound covers it."
  - "D2 and acceptance criteria: settle whether the clearing rule (begin-batch or recover-batch opens a new bracket) is stated for every path that opens one, and whether the criteria pin both the carried-over case and the cleared case."
  - "D1 (attack surface): the hex-only 4-to-39-character resolution at three consume sites. Settle whether a resolved commit that is neither HEAD nor reachable through the merge-only walk still meets the existing refusal, and whether recover-batch and `reset --to report-head` are bounded the same way as record-batch."
  - "D3 (attack surface): the derived-frame fallback relaxes today's refusal for states without `PlanOverviewFrameHash`. Settle whether the bound holds when `PlanFileHashes` names a baseline copy that differs from the file the run actually planned against."
  - "D5 against D2: D5 rewords the `report unattributable` row while D2 keeps zero transcripts as `ErrNoForkTranscripts` as today. Settle whether the rewording has a stated new trigger or the two disagree."
  - "D4 and Open risks: settle whether the derivation (start context minus reconstructed text lines times `context_per_line`) is unambiguous for the plan writer, and whether the skill-load risk is a design gap or a bounded risk."
  - "Relocation: D2 and D3 rationales carry session ids, commit hashes and replay narrative. Apply the completeness-before-leanness test first: keep any requirement or constraint in `decision-record.md`, and move only deliberation narrative to `support-log.md`."
---

Seed pass only; no prior round exists.
None of the entries targets a missing "Notes for the plan writer", rejected alternatives in the decision record, or call-site enumeration, per the rubric's `Do not flag` list.
The discussion stencil forbids gathering exact signatures and `file:line` citations, so none is promoted as missing.
