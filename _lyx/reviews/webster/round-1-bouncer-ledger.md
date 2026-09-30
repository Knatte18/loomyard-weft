---
round: 1
ledger:
  - key: exclude-anchored-dir-doc-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: path-tracked-doc-mid-sentence-wrap
    rounds: [1]
    status: resolved
  - key: write-config-doc-new-sentences-wrapped
    rounds: [1]
    status: resolved
  - key: exclude-test-doc-comments-wrapped
    rounds: [1]
    status: resolved
  - key: open-error-test-filters-warnings-before-count
    rounds: [1]
    status: resolved
  - key: spawn-driven-tests-attach-task-label-only
    rounds: [1]
    status: resolved
  - key: innerrun-an-once-marker-article
    rounds: [1]
    status: resolved
  - key: already-ignored-test-gitignore-uncommitted
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so there is no prior ledger to carry forward.
The reviewer raised eight findings against the diff: four comment line-break findings (F1–F4), two test-strength findings (F5, F6), a wording NIT (F7) and a test-setup NIT (F8).
The fixer fixed all eight in separate commits, and each fix holds on HEAD, so every entry is resolved.
The plan's full `verify:` block passes on HEAD.
The judge's own read of the diff found no new defect.
The only residue is a clause-level break before ", and returns" in `ExcludeAnchoredDir`'s first doc sentence, which is below the finding bar and has no ledger entry.
