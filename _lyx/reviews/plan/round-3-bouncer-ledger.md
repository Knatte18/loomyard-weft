---
round: 3
ledger:
  - key: refusal-spec-rows-missing
    rounds: [1]
    status: resolved
  - key: docs-constraint-package-docs
    rounds: [1]
    status: resolved
  - key: byte-identical-prompt-unpinned
    rounds: [1]
    status: resolved
  - key: await-cap-spelled-twice
    rounds: [1]
    status: resolved
  - key: fork-edge-bound-incomplete
    rounds: [1]
    status: resolved
  - key: twill-spec-test-duplicates
    rounds: [1]
    status: resolved
  - key: await-verb-noarg-row-duplicate
    rounds: [1]
    status: resolved
  - key: remove-ready-marker-test-duplicate
    rounds: [1]
    status: resolved
  - key: twilldirective-empty-field-refusals
    rounds: [1]
    status: resolved
  - key: ready-marker-accessor-card-cut
    rounds: [1]
    status: resolved
  - key: file-header-edits-outside-targets
    rounds: [1]
    status: resolved
  - key: env-seam-fixtures-missed
    rounds: [2]
    status: resolved
---

Round 1 raised eleven findings, led by a BLOCKING one about missing refusal-spec rows; round 2 did not re-raise any of them.
Round 2 raised one BLOCKING `scope` finding: card 10 added a required `Env` seam without targeting `FullEnv` or battencli's `intentionallyNil` table.
Round 3 found no findings; card 10 now lists both targets, so every key is resolved.
