---
round: 1
focus:
  - "Whole diff, every file: confirm each changed line is a comment or blank line and each changed file is a doc.go; a non-comment change or a non-doc.go file breaks the 'no code changes, other comments out of scope' rule."
  - "Largest cuts (fabricengine, websterengine, gatecli, reedengine, shuttleengine, tenderengine): does each doc.go still carry a short overview, its contracts and invariants, and the why the code does not show, or did the trim delete a guarantee, refusal or never-does statement no exported symbol's doc carries?"
  - "Cross-references: do headings, invariant names or sentences that other files, specs, PATTERN entries or code comments cite (grep for the removed text and for invariant names) still exist in the trimmed doc.go? A trim that orphans a citation breaks the answer key's 'what stays' rule."
  - "Commit message of 20ad9235 claims headings, invariant names and cited sentences stay and that PATTERN restatements now name the entry: check that each named PATTERN entry exists, and spot-check the claim against the diff."
  - "Residual superfluous text: sample doc.go files the diff changed little or not at all (yamlengine, toolfloor, small packages) and the trimmed large ones for remaining step-by-step narration, history, enumerated consumers or examples the operator decision lists as superfluous."
  - "Package doc comment validity: does each trimmed doc.go still open with a 'Package <name>' comment attached to its package clause, with no stray text or build-constraint change, and does any file now consist only of a package clause where a doc was expected?"
---

Comment-only change over 130 doc.go files.
No tests or code changed, so the reach, mutation and redundancy reports are empty.
Line wrapping is the comment lint's subject and is not a finding.
