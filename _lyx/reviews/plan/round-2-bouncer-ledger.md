---
round: 2
ledger:
  - key: card27-traced-glyph-regex
    rounds: [1]
    status: resolved
  - key: card11-loomshed-yaml-allowlist
    rounds: [1]
    status: resolved
  - key: card25-treesitter-link-guard
    rounds: [1]
    status: resolved
  - key: card4-removes-prescribed-counts-done-cards
    rounds: [1]
    status: resolved
  - key: round-start-whole-second
    rounds: [1]
    status: resolved
  - key: card4-line-unprescribed-frozen-card
    rounds: [1]
    status: resolved
  - key: card13-darn-refusal-order
    rounds: [1]
    status: resolved
  - key: card5-overviewat-empty-revision-row
    rounds: [1]
    status: resolved
  - key: card11-undecodable-record-way-forward
    rounds: [1]
    status: resolved
  - key: card12-unparsable-round-start
    rounds: [1]
    status: resolved
  - key: card13-plan-parse-failure-retry
    rounds: [1]
    status: resolved
  - key: card6-stale-seven-label-lists
    rounds: [1]
    status: resolved
  - key: card6-planparser-doc-deferred-to-card8
    rounds: [1]
    status: resolved
  - key: card13-amends-plan-shadows-no-record
    rounds: [1]
    status: resolved
  - key: card18-amendmenttarget-bool-signature
    rounds: [1]
    status: resolved
  - key: card19-amendment-target-read-once
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card24-targetholds-string-prefix-reading
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised fifteen findings.
Round 2 did not raise any of them again, and its Rejected section re-examined several and found no defect, so all are resolved.

Round 2 raised two new findings.
The card 19 finding is BLOCKING and gating: the arbiter's `amendment_target` is read at wiring time, so a drive that begins before plan approval never sees it.
Its fix adds an `AmendmentTarget func() string` to `arbiterDeps` read at each spawn, which changes a contract, so it is `design-wrong` and needs a fresh review.
The card 24 finding is a prose clarification that card 24 must read the owner chain from parsed glyphs, and its text fixes the correction.
