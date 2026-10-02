---
round: 1
ledger:
  - key: tier-rule-asymmetric-done-exemption
    rounds: [1]
    status: resolved
  - key: legacy-read-byte-identical-vs-swaplock
    rounds: [1]
    status: resolved
  - key: legacy-slug-refusal-and-fold-read-path-tests
    rounds: [1]
    status: resolved
  - key: retire-legacy-after-save-step
    rounds: [1]
    status: resolved
  - key: legacy-file-name-constant-declarers
    rounds: [1]
    status: resolved
  - key: card4-impactsummary-blast-radius
    rounds: [1]
    status: resolved
  - key: semantic-line-breaks-cards-6-10
    rounds: [1]
    status: resolved
  - key: card6-impactsummary-list-order-consumers
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings: two MEDIUM, five LOW and one NIT.
The MEDIUM findings were card 1's asymmetric `done` exemption in the tier rule and card 4's byte-identical read assertion, which a locked legacy read's swaplock creation contradicted.
The LOW and NIT findings were missing read-path tests, the RetireLegacy save/delete/render ordering mechanism, duplicate legacy file-name constants, two thin ImpactSummaries, and semantic line breaks not carried into cards 6 and 10.
The fixer addressed all eight without changing card targets or structure, and the judge confirmed each fix in the current text.
The plan was approved at round 1, with no open entries.
