---
round: 2
ledger:
  - key: round-gate-tagged-compile-gap
    rounds: [1]
    status: resolved
  - key: publish-verify-unbounded-config
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: lyx-guard-missed-marker-unbounded
    rounds: [1]
    status: resolved
  - key: comment-lint-flags-untouched-lines
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: comment-lint-terminator-rules
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: diff-mapping-rename-delete-baseless
    rounds: [1]
    status: resolved
  - key: empty-set-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: verify-record-retention-and-lookup
    rounds: [1]
    status: resolved
  - key: round-command-length-growth
    rounds: [1]
    status: resolved
  - key: tier-green-ordering-misplaced
    rounds: [1]
    status: resolved
  - key: decision-record-narrative-duplicates
    rounds: [1]
    status: resolved
  - key: publish-verify-no-publish-landings
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-llm-catch-claim
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-no-base-lint-premise
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d5-wrong-gate-catcher
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d8-guard-tag-set
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d5-guard-enumeration
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised eleven findings, gating ones on `publish-verify-unbounded-config` and `lyx-guard-missed-marker-unbounded`.
Round 2's fresh review did not raise the round 1 keys other than three: `publish-verify-unbounded-config` (F1, now BLOCKING because D4's replacement bound cites a PR diff that never carries weft config), `comment-lint-flags-untouched-lines` (F4, D6 and D7 disagree on untouched lines) and `comment-lint-terminator-rules` (F6, D6 omits closing-punctuation sentence ends).
The rest of round 1 is carried as resolved.
New in round 2: F2 (landings outside `require_pr_to_base` run no tmux gate, design MEDIUM), F3 (D2 claims Publish catches llm-only failures), F5 (D7's no-base lint premise), F7 (D5 names Webster's gate, which runs before the rounds), F8 (D8's tag set) and F9 (no way to enumerate existing guards).
