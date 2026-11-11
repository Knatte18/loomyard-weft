---
round: 2
focus:
  - "internal/loomcli/decisionreplace_test.go TestDecisionReplaceVerb_Refusals and TestDecisionReplaceVerb_CommitOutcomes: does every refusal that decisionReplaceVerb and landReplace return, other than a propagated raw I/O failure, have a row that reaches it and asserts its message and way forward, as PATTERN-refusal-way-forward requires?"
  - "internal/loomcli/decisionreplace_test.go TestDecisionCalls_RefreshEveryStoredRecord: does a row reach the record.project exclusion in boardRecordStored, so that add on a beam run leaves the project entry's stored record unchanged and the row would fail if the exclusion broke?"
  - "internal/loomcli/decision.go store command Long text against decisionStoreVerb: does the help state the admission past a missing Discussion approval when the entry carries a stored record, and the refusal of a raddle run whose filing is done?"
  - "contracts/specs/loom-discussion-spec.md, parts no writer authors: does the spec name both replace and lym board upsert --record-file, and say when each is used?"
  - "internal/discussionparser/replacecheck.go CheckAddedEntriesSubset: does a file that repeats an existing added entry fail, and does the byte-for-byte comparison hold given the trailing-whitespace trim?"
  - "docs/overview.md around line 814, contracts/stencils/loom/loom-directive-project-record.md, contracts/recipes/pawl-recipe.yaml: do the beam project-record texts name replace alongside decision add, as the declared PATTERN-run-records text does?"
  - "internal/loomcli/decisionreplace_test.go TestDecisionLock_SerializesRecordWriters: does a row pin that decisionStoreVerb takes and releases the decision lock and refuses when the lock fails?"
  - "internal/loomcli/recordhistory_test.go TestRecordHistoryCut: is the whole-history fallback when no cut is found reached through recordHistory itself rather than through an inline copy of its glue?"
---

Each entry names a site and the question to settle; none caps severity or pre-states a verdict.
