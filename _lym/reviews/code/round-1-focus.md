---
round: 1
focus:
  - "internal/loomcli/decision.go replace: does every refusal D1 lists (base hash, Discussion approval row, Webster span, filed raddle run) name a way forward, and does the lock cover add, replace, store and writeCarryOver from the first precondition through commit and board copy?"
  - "internal/loomcli/decision.go commit-error path: does it follow D1's order (HEAD holds bytes with trailer, HEAD holds bytes without trailer, HEAD lacks bytes and prior bytes are restored), with the right refusal text for each?"
  - "internal/loomcli/projectrecord.go recordHistory and internal/fabricengine seams: is the cut ordered by first-parent position, never committer time, and does it check each trailer's own path and fall back to the whole history when the sha256 matches no version?"
  - "internal/discussionparser/replacecheck.go: do the carry-over equality and added-entry subset checks compare byte for byte, and can any dropped, edited or inserted entry slip through?"
  - "D3 board copy: is the stored-record refresh gated as specified for raddle runs, beam runs and runs re-entered into Discussion, and does the filed refusal cover the three filing-done cases?"
  - "Fabric and gitrepo boundaries: does CommitEmptyWithTrailers commit the record path alone, allow an empty commit, and leave other staged content staged?"
  - "contracts/specs, the parent directive, docs/overview.md and PATTERN files: do they carry each wording the D6 and Review acceptance criteria ask for?"
  - "Test economy: do the new tests in decisionreplace_test.go, recordhistory_test.go and the integration tests overlap one another, and does each behavior the diff adds have coverage?"
---

Focus covers the answer key's riskiest decisions: lock atomicity, the commit-error ladder, the history cut and the subset checks.
