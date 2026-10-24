---
round: 1
focus:
  - "Amendment scope: the parent's changes to the 22 overlap-note files (test files under cmd/lyx, contracts/stencils, internal/lyxcwd, internal/planparser, cmd/testtiming/redundancy.go, and the four pattern/PATTERN-test-*.md and testkit files) after the merge. Settle whether the merged test-speed, test-isolation and test-economy content keeps the parent's new text and every test message cites a PATTERN ID that still exists."
  - "The two conflicted files, pattern/PATTERN-test-economy.md and pattern/PATTERN-testkit.md: both are removed in the diff. Settle whether anything the parent added to them since ed98ccf71 landed in the survivors test-speed or test-isolation, with nothing dropped."
  - "Cited PATTERN IDs: no test checks that a cited ID exists. Settle whether any .go file, contract spec, crucible file or doc still cites a removed or merged ID, including IDs the parent's new commits introduced (gate-runs-once tier-tag and verified-tree tests)."
  - "Mechanics of internal/pattern.Check: every pattern/ file is named by exactly one entry. Settle whether PATTERN.md entries and pattern/ files match one to one after the merge, and whether the dropped and moved entries (board entry's Move and Drop lists) each landed in the stated home (doc.go or dropped for an enforced home)."
  - "Moved rules in doc.go (for example tools/sandbox/doc.go): settle whether each moved rule keeps its meaning against the original background file, and whether docs/overview.md and CLAUDE.md stay consistent with the new entry set."
  - "Merge groups in the board entry: settle whether each survivor absorbed its listed entries with carve-outs kept (for example the lyxdirs-single-declarer tools/ carve-out under durable-vs-ephemeral-state), and whether the diff merges or moves anything the board entry does not list."
  - "Size goal: settle whether PATTERN.md ends well under its headroom line, and whether the decision record's Added after Discussion entries amend any board-entry list the diff follows or departs from."
---

Seed focus for the amendment round; each entry names where to look and the question to settle.
