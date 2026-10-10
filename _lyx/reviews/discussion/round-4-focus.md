---
round: 4
focus:
  - "D2 in _lyx/discussion/decision-record.md rejects unquoted glob characters before `--` so a planted file named like a writing flag cannot act as a flag, and its Bound says the classifier assumes bash or zsh with any glob option set; settle whether the rule is closed by construction (for example, rejecting any unquoted word that is not a plain literal) or by an enumerated character list, and whether every zsh EXTENDED_GLOB form that can match a name whose first character differs from the word's, including `#` and `##`, is rejected or recorded as a bounded residual."
  - "D6 in _lyx/discussion/decision-record.md has a dead-fork-record case with acceptance rows; check against internal/websterengine (recordbatch.go, digest.go, batchfail.go, classify.go) and internal/websterengine/beginbatch.go whether any code path gives a fork record the dead status, and whether the case and its rows are removed or explicitly marked defensive."
  - "D6's exhausted text for a stuck or failed record at the recovery cap names `reset --to batch-start` then `recover-batch`; check it against contracts/stencils/webster/webster-template-master.md, whose begin-batch rung follows the printed remedy and whose rule allows no reset but `report-head`, and settle whether begin-batch's refusal carries a `recovery_exhausted` envelope flag, whether Scope names that CLI change, and whether D6's claim that begin-batch and Merriam never disagree covers this case."
---

Round 3 left these three areas as the only open questions; fresh text added to answer them is new and unseen, so read each changed Decision and its Acceptance rows as a whole.
