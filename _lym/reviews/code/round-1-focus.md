---
round: 1
focus:
  - "internal/planglyph/donecheck.go `signatureMismatches` and `doneCheckVerdicts`: does the mismatch finding fire only when the unit declares the member under the same owner and name with other parameter lists, and never when a declaration with the exact wanted parameter list exists (for example in an overloaded or counterpart-package case)? Is the order against `declaredInCounterpartPackage` right, and does the Rename new side behave the same as Create?"
  - "internal/planglyph/donecheck.go `declarationList` and `searchedDescription`: does the cap of three with an 'and N more' count hold for every path, is the nearest-first order deterministic, and is the unit file count right when the unit has zero, one or many files? Does the issue #63 requirement hold: unit files stated once per unit rather than per target, with the one-line per-target finding first?"
  - "internal/websterengine/recordbatch.go `createTargetWayForward`, `failCardNotDone` and `cardNotDoneFor`: does recover-batch refusing the same create findings name a way forward other than recover-batch itself (issue #62)? Check the precedence against the delete-not-done and glyph-rejected way forwards when findings mix, the `in.Verb == \"recover-batch\"` test for 'a recovery already tried', and the uncheckable-record branch."
  - "Issue #61 coverage against the diff: signature-mismatch leads with the declared signature and file, the searched-file list is capped or replaced by a count, and the corrective step (amend the card's target signature) sits next to the recover-batch hint. Is anything the three issues ask for left unmade, or anything made that they do not license?"
  - "Claims in _lym/landing/summary.md against the diff: the example detail strings, the claim that Go glyphs never carry parameters, and the claim that a rebaseline amendment makes the next recovery spawn uncounted."
  - "Tests and docs: does the new test duplicate coverage in `TestDoneCheckVerdicts_Rules` or `TestDoneChecks_Verdicts`? Is any behavior the diff adds uncovered, in particular the `signature-mismatch` branch of `failCardNotDone`? Do the doc comment and doc.go changes in internal/planglyph and internal/websterengine, and the status tripwire allowlist in status_enforcement_test.go, match the repository's own rules?"
---

Seed pass: the review range is `4c71d798..HEAD`, four commits touching `internal/planglyph` and `internal/websterengine`.
The answer key is the decision record's `## Goal`, which carries issues #61, #62 and #63; the record has no plan.
