---
round: 1
focus:
  - "contracts/stencils/burler/burler-template-fixer.md: the read-together sentence sits in step 2 (Validate) and the apply sentence in step 3; settle whether each stencil change matches the board entry's fixer sentence (gather files, edit independent changes in one message, gate once after the last edit) and drops no existing fixer rule."
  - "contracts/stencils/darn/darn-template-write.md Step 4 items 1 and 2: settle whether the gather, batched-edit and gate-once-per-commit wording matches the board entry's rule, keeps the 'each commit leaves the tree green' and 'never a raw test command or module-wide run' rules, and uses 'message' not 'turn'."
  - "contracts/stencils/webster/webster-body-implementer.md step 2 and step 3: settle whether the card's gate stays per card, run by the step 3 gate rule, and whether the new sentence's 'run the card's gate once' contradicts step 3's step-by-step gate list."
  - "internal/shuttleengine/claudeengine/settings.go and prepare_test.go / settings_test.go: settle whether the appended prompt orders deny notice, creel sentence, batching sentence with single-space separators in both the deny and no-deny cases, that the batching sentence text equals the board entry's sentence exactly, that the init line-break check covers both sentences, and that the tests cover each new behavior without duplicating existing assertions."
  - "contracts/stencils/burler/burler-step-1-explore.md: the sentence follows the `{{.tool_use_rules}}{{.fork_reading}}` line; settle whether it keeps the `fork_reading` marker position that contracts/stencils/markers_test.go pins, and reads correctly in a solo round and a fork round."
  - "docs/overview.md and the claudeengine doc.go: settle whether the shuttle module entry and the package doc describe the file's contents as the code writes them, and whether any other doc or comment that names the creel sentence as the file's only second content is left stale."
---

Seed pass: focus is set from the board entry, the decision record and a read of the diff.
All eight stencils and the engine change are in the diff; no PATTERN change is declared.
