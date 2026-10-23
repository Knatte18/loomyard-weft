---
round: 3
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
    status: resolved
  - key: card24-targetholds-string-prefix-reading
    rounds: [2]
    status: resolved
  - key: card19-arbiter-doc-go-stale
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card24-file-rule-path-reading-unnamed
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised fifteen findings, and round 2 did not raise any of them again, so all are resolved.

Round 2's card 19 BLOCKING finding, the arbiter's `amendment_target` read at wiring time, was fixed with a per-spawn `AmendmentTarget func() string`.
Round 3 re-examined it, rejected it as no defect, and the key is resolved.
Round 2's card 24 string-prefix finding is likewise rejected in round 3 apart from a narrower gap, which is filed as a new key.

Round 3 raised two new findings.
F1 is MEDIUM scope: card 19 changes the arbiter's observable behavior without editing the Arbiter section of `internal/loomcli/doc.go`; the fix adds the file to the `Edit` list and a paragraph to the Intent, so it is mechanical.
F2 is LOW consistency: card 24's file rule does not name `PathEntryPath` and `HandleBody`; the fix adds two `Uses` entries and one Intent sentence, so it is mechanical.
