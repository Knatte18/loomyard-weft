---
round: 2
ledger:
  - key: card4-card-packs-column-contradicts-test-row
    rounds: [1]
    status: resolved
  - key: card4-card-arm-spans-biased-window
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card2-standalone-runs-lose-pack-default-on
    rounds: [1]
    status: resolved
  - key: card2-empty-card-test-row-conflates-cases
    rounds: [1]
    status: resolved
  - key: card2-stale-packcard-doc-comment
    rounds: [1]
    status: resolved
  - key: card4-net-columns-omit-card-pack-calls
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: card4-pack-check-grades-later-card-spans-against-start
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card3-fragment-banners-name-only-pack-file
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card2-wrong-mechanism-for-non-hub-refusal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card1-operator-comments-omit-hub-only-limit
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-mainforks-count-sentence-and-fork-card-timestamp
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-disagreement-rule-zero-call-case
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings; round 2 reports them fixed, bar the card-arm span window.
That one is reopened: the card arm's per-card window now exists, but the report labels both windows alike and discloses the difference only in jq source.

Round 2 adds seven keys.
Two are design-adjacent: card-arm pack-check grades card 2+ spans against the batch start sha, and the net columns leave the card-pack calls in for the card arm only.
The rest are local text corrections on cards 1 to 4: four fragment banners, the non-hub refusal mechanism, operator comments for the new key, the `mainForks` count sentence and fork-card timestamp, and the zero-call disagreement wording.
