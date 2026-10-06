---
round: 3
focus:
  - "Decision 1 keeps a candidate that a core or testkit file names, and Decision 10 requires a `//testtiming:keep <reason>` on every candidate the rerun still lists while forbidding annotation of a candidate with nothing unique to pin. Settle whether every listed candidate has exactly one permitted outcome (delete, fold, or keep with a directive) and whether each keep reason class, including a core or testkit reference, is named in Decision 10."
  - "Decision 2 moves a unique assertion from a deleted candidate into a covering test, and Decision 10 repeats the rerun until the list is clean. Settle whether the receiving test is protected from later deletion in the repeat loop, and whether, if the rerun lists it, a permitted keep reason fits it."
  - "Decision 5 leaves an integration test in place when a tier move needs a shared fake that does not exist. Settle where the reason for staying lives so that a later pass over the same tests finds it, not only in the PR body."
  - "Decision 12 defines a coverage rule under `-coverpkg=./...`. Settle whether the per-package figure is defined precisely (which files, which profile), and whether the rule for a drop is consistent with the rationale that removable deletions leave every block covered."
  - "Decision 9 and Decision 10 both describe rerunning the timing tool, and Decision 9 splices per-package output outside the tool. Settle who performs the splice, how often the rerun runs and over what scope, and what happens to a package that ends with no tests."
  - "Decision 2 requires the PR to list a receiving test for each deleted candidate whose assertion moved. Settle whether an acceptance criterion checks that list."
---

Entries restate the decisions they depend on, so the reviewer needs no earlier round's material.
